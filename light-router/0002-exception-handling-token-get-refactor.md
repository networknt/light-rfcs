# Router Exception Handling

The current router needs to be refactored to:

Support multiple auth server.

Support handling error scenario.

## Support multiple auth server

**For successful Access Token Response**, different auth server should contain same information as per [https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/](https://www.oauth.com/oauth2-servers/access-tokens/access-token-response/)

- `access_token` (required) The access token string as issued by the authorization server.
- `token_type` (required) The type of token this is, typically just the string “bearer”.
- `expires_in` (recommended) If the access token expires, the server should reply with the duration of time the access token is granted for.
- `refresh_token` (optional) If the access token will expire, then it is useful to return a refresh token which applications can use to obtain another access token. However, tokens issued with the implicit grant cannot be issued a refresh token.
- `scope` (optional) If the scope the user granted is identical to the scope the app requested, this parameter is optional. If the granted scope is different from the requested scope, such as if the user modified the scope, then this parameter is required.

Thus, one ResponseToken should handle different auth server responses.

**For failure Access Token Response**, different auth server may use different response keys to describe error status.

Thus, we need a generic way to handle error scenarios so that we can send response info from auth server to the client side.

## Support handling error scenario

- Refactor OauthHelper.getToken()
    - change return type from TokenResponse to Result<TokenResponse>, Result is com.networknt.monad.Result. In this case, we don't need to throw exception to the ExceptionHandler anymore.
    - when success, this Result will contain TokenResponse info:

            result = Success.of(tokenResponse);

    - when fail, this Result will contain Status info, (defined in status.yml):

            result = Failure.of(new Status(GET_TOKEN_ERROR, responseBody));

- Add a @FunctionalInterface to define callback function when fail

        private void getCCToken(FailureHandleable callback) {
                Result<TokenResponse> result = getCCTokenResponse();
                if(result.isSuccess()) {
                    TokenResponse tokenResponse = result.getResult();
                    jwt = tokenResponse.getAccessToken();
                    // the expiresIn is seconds and it is converted to millisecond in the future.
                    expire = System.currentTimeMillis() + tokenResponse.getExpiresIn() * 1000;
                    logger.info("Get client credentials token {} with expire_in {} seconds", jwt, tokenResponse.getExpiresIn());
                } else {
                    callback.handle(result.getError());
                }
            }

- Refactor TokenHandler.checkCCTokenExpired()

        boolean isInRenewWindow = expire - System.currentTimeMillis() < tokenRenewBeforeExpired;
        logger.trace("isInRenewWindow = " + isInRenewWindow);
        if(!isInRenewWindow) { return; }
        synchronized (TokenHandler.class) {
            //if token expired, try to renew synchronize
            if(expire <= System.currentTimeMillis()) {
                //the token can be renew when it's not on renewing or current time is lager than retrying interval
                if (!renewing || System.currentTimeMillis() > expiredRetryTimeout) {
                    renewing = true;
                    expiredRetryTimeout = System.currentTimeMillis() + expiredRefreshRetryDelay;
                    getCCToken((status) -> {
                        //set renewing flag to false when fail
                        renewing = false;
                        //on get token failure, send response back according to status
                        OauthHelper.sendStatusToResponse(exchange, status);
                    });
                    //set renewing flag to false when success
                    renewing = false;
                }
            }else {
                // Not expired yet, try to renew async but let requests use the old token.
                logger.trace("In renew window but token is not expired yet.");
                if(!renewing || System.currentTimeMillis() > earlyRetryTimeout) {
                    asyncRenewCCToken();
                }
            }
        }

    Something we need to discuss is the ***synchronized*** keyword, the previous code is using Double-checked locking ******to check "if(expire <= System.currentTimeMillis())", may accelerate the process. We need to discuss how big differences between these two.

- For Http2Client Module, there are many duplicate code comparing to Router module, should we refactor two places with duplicate changes? Or we need to rethink about it?

