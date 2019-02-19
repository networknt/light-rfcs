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

The JWTs should be cached based on scopes, if the scopes provided have same elements but different order, they should be considered as same scopes and will retrieve the same JWT.

I think we also need to set a capacity of cache and use LRU policy, because with a 100 scopes for a service, it may have huge combination.

By now, the TokenHandler in router module and Http2Client in client module cannot be used together for Auth usage, it's the one or the other. (input by [@NicholasAzar](https://github.com/NicholasAzar), correct me if I'm wrong). I'm thinking instead of caching JWT in each module, should we create a singleton JWT Manager to manage JWTs globally, in case of other module may need JWT? (Although I haven't thought out any use cases yet.) But to use a JWT Manager will give some benefits for example to clean up those expired JWT caches, instead of each module managing their own JWT caches.

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
4. Renewing JWT2 shouldn't affect renewing JWT1. We should start to "populateCCToken()" of JWT1.
5. Suppose there comes another request with scopes: [eat, drink], it will get cached JWT2, and find the it's expired (at this time, step 2 is not finished yet, so the token is still shown as expired), then it calls "populateCCToken()" of JWT1. But it should be blocked by the Thread lock of the JWT1 object to avoid duplicate renewing.

Thus, we need to change Thread lock from class level to object level. 

public static Result<Jwt> populateCCToken(Jwt jwt) {<br/>
    synchronized (~~OautherHelper.class~~ jwt) {<br/>
    ...<br/>
    }<br/>
}<br/>
