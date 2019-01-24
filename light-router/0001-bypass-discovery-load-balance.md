### Summary
The router's current behavior is to use service discovery or local config to decide which service instance a request message should be forwarded to.
This change is to add a new http header to allow the caller of the router to control where the request should go.
Behaviors of the requests that do not have new header should not be affected by this change.
There is a restriction about the hosts of which services that can be used in the new header. They are defined in the router config (route.yml) with the config key "hostWhitelist".

### Motivation
This is from a Jira ticket: API-36: Router support to bypass service discovery & balancing.
There is a usecase that requires the caller of the router to determine which service instance the request should be forwarded to. Instead of service discovery, callers provide the service url in the http header. In this case, the router should ignore the service id (if provided) and forward the request to the service.

### Guide-level explanation
To use this feature, callers of the router need to set the key, "service_url" in the header of the request. Its value should be a valid url. For an example:
```
https://10.1.2.3:8543/
```
If the "service_url" does not exist in the request header, the router behaves like before which goes to either service discovery or local config (service.yml) to determine where to go.

As part of the router configuration, the key, "hostWhitelist" must be configured in the router.yml to control the valid host patterns if the services that can be forwarded to. Otherwise the request will failed unless "service_url" is not in header.

### Reference-level explanation
The "service_url" in the request header must be a valid url, otherwise the transaction will fail.
To enable this new feature in router configuration, the key "hostWhitelist" must be configured in the router.yml. The following is an example.
```
hostWhitelist:
  - localhost
  - 192.168.0.*
```
This setting is an array of strings. Each of them is a regular expression to specify the valid host name. The host part of the service url in the request header must match at least one of these host name patterns. Otherwise, the request will be rejected.

This hostname validation only be turned on when the income request carries the new "service_url" header.
The "hostWhitelist" setting can be optional if the service url header is not provided.

### Drawbacks
Caller of the router service must know the exact service url.

### Rationale and Alternatives
The router will forward the incoming requests based on the following criterias:
* Service url
If the key, "service_url" is provided in the header, the request will be forwarded to its value. The service id header will be ignored in this case.
* Service id
If the service url is not provided, but this key, "service_id" is, the router will continue with the original behavior which goes with service discovery (or as local config) to determine the which service to forward to.

### Unresolved questions
Error handling related to undelivered requests may need to be reviewed in terms of status code and messages.
