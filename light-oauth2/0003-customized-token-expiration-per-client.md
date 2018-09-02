### Summary

In light-oauth2, there is only one place in jwt.yml that you can config the JWT token expiration time and all tokens issued will use the same config. In most of the cases, it would be set as 10 minutes to 15 minutes. 

It is a security risk to have long lived JWT token because there is no way that we can revoke an issued token. That is why most OAuth 2.0 providers will set a default expiration time as 10 to 15 minutes. 

In our reference architecture, we are encouraging users to use authorization code flow with refresh token, and the framework handles all the token refresh automatically. For a long-running process, a short-lived token is fine as it is used only for the initial call unless your long running process is calling other APIs with that original token regularly during the live period. That is a little bit complicated to implement in the batch process. In this case, the workaround would be refresh token, but it makes user application more complicated. 

### Motivation

For the above batch process scenario, it makes sense to give that particular client a customized expiration time so that the token can last the entire life-cycle of the batch job. This will make the batch job developer's life much easier. As this is mostly usesd internal so the risk of long-lived token is limited. 

### Guide-level explanation

The light-oauth2 expands the OAuth 2.0 specification and added trust client concept. That means that the OAuth 2.0 provider trust the client which is in the same organization and even in the same data center. 

A new column will be added to the client table for the token expiration overwrite. When token is created, the token service first check if the client is trusted. If yes, then check if there is a customized expiration time to be used. 

The client registration endpoint needs to be updated to add this new column and the validation rule must be added to ensure that only trusted client type allows to do that. 

### Reference-level explanation

Internally, the cached client object needs to be updated to add this new column. 

### Drawbacks

It takes an extra cycle to check the client type and customized expiration time for token service but the performance impact should be rather small. 

### Rationale and Alternatives

Alternative solution would be ask the user to leverage the refresh token for authorization code flow and client credentials flow. The logic of use refresh token must be built by the user in the batch process. 

### Unresolved questions

Is this use case a typical and should we get it implemented?
