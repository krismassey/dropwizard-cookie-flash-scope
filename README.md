# Dropwizard cookie-based flash scope


A flash scope for Dropwizard that avoids the need for any server-side state. Useful when you're running multiple nodes
but don't want the complexity of sticky sessions or an external session store.

Note: the flash cookie is not encrypted or signed in anyway, so don't use it to store any confidential data, or anything
where a user modifying the contents of the cookie might cause problems.


## Adding to your application

### Default configuration
If you're happy for the flash scope cookie to be created with the default configuration, add this to your Application
class:

```java
@Override
public void initialize(Bootstrap<MyAppConfig> bootstrap) {
    bootstrap.addBundle(new FlashScopeBundle<MyAppConfig>());
}
```

This will cause the flash scope to be managed in a cookie called ```DW_FLASH``` with a 5 second timeout, a path
of ```/```, on the request domain.

### Configured
If you want to override any of these settings you'll need to add an instance of ```FlashScopeConfig``` to your Dropwizard
config class e.g.

```java
@JsonProperty
private FlashScopeConfig flashScope;

public FlashScopeConfig getFlashScope() {
    return flashScope;
}
```

Add the relevant config to your YAML file:

```yaml
flashScope:
  cookieName: MY_FLASH_SCOPE
  cookiePath: /my-app
  cookieMaxAge: 20s
  cookieDomain: flashexample.com
```

And initialise it in your application like this:

```java
@Override
public void initialize(Bootstrap<MyAppConfig> bootstrap) {
    bootstrap.addBundle(new FlashScopeBundle<MyAppConfig>() {
        @Override
        protected FlashScopeConfig getFlashScopeConfig(MyAppConfig configuration) {
            return configuration.getFlashScope();
        }
    });
}
```

## Using in your app
The flash scope is made available to your web resources via an injected ```Flash``` parameter. The ```Flash``` class
has generic ```get``` and ```set``` methods which return/take any object serialisable by Jackson (using Dropwizard's
```ObjectMapper``` instance).

e.g. to define an action that populates the flash and redirects to a view:

```java
@Path("/action")
@POST
public Response doSomething(@FlashScope Flash flash, @FormParam("message") String message) {
    flash.set(new FlashMessage(message));
    return Response.seeOther(URI.create("/result")).build();
}

@Path("/result")
@GET
@Produces("text/plain")
public String getResult(@FlashScope Flash flash) {
    Optional<FlashMessage> contents = flash.get(FlashMessage.class);
    return (contents.or(new FlashMessage("OK"))).getMessage();
}
```


