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

- Refactor TokenHandler.checkCCTokenExpired()
    - change method name to populateCCToken
    - instead of modifying class member directly, should populate the given JWT info passed as parameters
    - move this method to OauthHelper and make it static so that this part can be used in multiple modules(Http2Client, TokenHandler)

            public static Result<Jwt> populateCCToken(Jwt jwt) {
                boolean isInRenewWindow = jwt.getExpire() - System.currentTimeMillis() < jwt.getTokenRenewBeforeExpired();
                logger.trace("isInRenewWindow = " + isInRenewWindow);
                //if not in renew window, return the current jwt.
                if(!isInRenewWindow) { return Success.of(jwt); }
                //block other getting token requests, only once at a time.
                //Once one request get the token, other requests don't need to get from auth server anymore.
                synchronized (Http2Client.class) {
                    //if token expired, try to renew synchronously
                    if(jwt.getExpire() <= System.currentTimeMillis()) {
                        Result<Jwt> result = renewCCTokenSync(jwt);
                        if(logger.isTraceEnabled()) logger.trace("Check secondary token is done!");
                        return result;
                    } else {
                        //otherwise renew token silently
                        renewCCTokenAsync(jwt);
                        if(logger.isTraceEnabled()) logger.trace("Check secondary token is done!");
                        return Success.of(jwt);
                    }
                }
            }

    - The whole process is synchronized to prevent multiple requests to get token at the same time.
    - Pass error Status through return type com.networknt.monad.Result, instead of throwing exceptions

a issue may be related to this router change is:  StatelessAuthHandler [light-4j/#322](https://github.com/networknt/light-4j/issues/322)
## Before change vs After change

|                      Cases                      |                                               Response/Behaviors                                               |                                                                    Message                                                                    | Message                                                                                                                                                                                                                      |
|:-----------------------------------------------:|:--------------------------------------------------------------------------------------------------------------:|:---------------------------------------------------------------------------------------------------------------------------------------------:|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|                Connection Refused               |                                             400: Uncaught Exception                                            | { "statusCode": 500, "code": "ERR10010", "message": "RUNTIME_EXCEPTION", "description": "Unexpected runtime exception", "severity": "ERROR" } | { "statusCode": 401, "code": "ERR10053", "message": "ESTABLISH_CONNECTION_ERROR", "description": "Cannot establish connection for url ${url}", "severity": "ERROR" }                                                         |
| Connection Established, but fail to send reques |                             500: Unexpected Runtime Exception fail to send request                             |                                                                                                                                               | { "statusCode": 401, "code": "ERR10054", "message": "GET_TOKEN_TIMEOUT", "description": "Cannot get valid token, probably due to a timeout.", "severity": "ERROR" }                                                          |
|    wrong client_id/client_secret combination    | NullPointerException (can receive response from layer7 but layer7 doesn't response a JWT but an error message) | { "statusCode": 500, "code": "ERR10010", "message": "RUNTIME_EXCEPTION", "description": "Unexpected runtime exception", "severity": "ERROR" } | {"statusCode":401,"code":"ERR10052","message":"GET_TOKEN_ERROR","description":"Cannot get valid token: { "error":"invalid_client", "error_description":"The given client credentials were not valid" }.","severity":"ERROR"} |
