# Router/Client Accepting Scopes

## Purpose

Add a feature to the client module and the router module so that the consumer can add scopes to  their requests to get a JWT from OAuth server. We need to provide the following ability:

### Receiving scopes:

In Router, it needs to get scopes info request headers with key "scopes", then pass it to the method that can populate client credential token (populateCCToken method).

In client module, it needs to add overload methods with an extra parameter "scopes" to existing methods that needs to populate client credential token (call populateCCToken method).

If scopes is not provided in requests, we should get it from config: client.yml
```
oauth:
  # OAuth 2.0 token endpoint configuration
  token:
    # the following section defines uri and parameters for authorization code grant type
    client_credentials:
      # optional scope, default scope in the client registration will be used if not defined.
      scope:
        - scope.A
        - scope.B
```
### Caching multiple JWTs:

Currently in Router/Http2Client, it's caching only one JWT, so we need to change it to cache multiple JWTs.

The JWTs should be cached based on a Key. The Key should contains scope info, service id. The key should be extendable which will support future use case. 

If the scopes provided have same elements but different order, they should be considered as same scopes and will retrieve the same JWT.

The scope in Key may be different from scope in a Jwt in the following scenario:

- The scopes contain a scope which is not registered in Oauth Server, the server will return a subset of the provided scopes.
- The scopes is not provided, the server will return all the registered scopes, or needed scopes which calculated by Oauth server.

We need to identify a capacity, it can be either fixed or configurable.

The caching policy should be easily extendable based on different strategy. For default, it's more reasonable to remove based on expiry time: when meet the capacity, the Jwt with least expiry time should be replaced. After renewing a cached token, the expiry time will be reset, and the priority of removing this token is also changed.

Use a Token Manager to manage all the tokens of different modules. The consumer should only call Token Manager to get a token, and the token manager will do the cache based on different cache strategy.

### Get Token Synchronously based on JWT:

The method to get/renew JWT should be object level sync based on JWT object.

For example, currently we have two cached scopes:
```
{
  "scopes":
  [
    {
      "key": [eat, drink, sleep],
      "value": JWT1
    },
    {
      "key": [eat, drink],
      "value": JWT2
    }
  ]
}
```
1. When there's a request with scopes: [eat, drink], it will get the cached JWT2 and check if JWT2 is already expired or not. Let's say it's already expired and needs to be renewed.
2. Then we start to "populateCCToken()" of JWT2, make a connection with the OAuth server, and preparing to send request to it.
3. At this time, there comes another request with scopes: [eat, drink, sleep], it will get the cached JWT1 and check if JWT1 is already expired or not. Let's say JWT1 is also expired and needs to be renewed.
4. **Renewing JWT2 shouldn't affect renewing JWT1**. We should start to "populateCCToken()" of JWT1.
5. Suppose there comes another request with scopes: [eat, drink], it will get cached JWT2, and find if it's expired (at this time, step 2 is not finished yet, so the token is still shown as expired), then it calls "populateCCToken()" of JWT2. But **renewing should be blocked by the Thread lock of the JWT2 object to avoid duplicate renewing**.

Thus, we need to change Thread lock from class level to object level. 

    public static Result<Jwt> populateCCToken(Jwt jwt) {
        synchronized (jwt) {
        ...
        }
    }

### The order of getting a token:

1. Based on provided scopes from consumer.
2. Based on default scope config defined in client.yml
3. Based on service id (suppose future Oauth server will return appropriate scope based on provided service id)

### Sending request to Oauth server:

To support different Oauth servers, the info to be ent should be the same, but the way to send the info may be different. Let's say the same info for example, scope, may be sent in request header for a Oauth server, but may be sent through query parameters in the other Oauth server. Thus, we need to provide an ability to support creating different ClientRequests based on different Oauth servers.
