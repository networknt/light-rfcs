### Summary

The current client module works very well for HTTP 2.0 server with HTTP 2.0 connections. However, we need to implement a connection pool for the use case that it is needed to connect to legacy API servers only support HTTP 1.1 protocol.

Also, we are relying on the caller to drop the connection after a certain number of requests or after certain minutes to force the client side load balancing. Need to build this into the module so that we can make this transparent for the developers. There is an issue opened for this at https://github.com/networknt/light-4j/issues/155

Another enhancement for the client module is to handle the retry gracefully at module level with configurations. 




### Motivation


### Guide-level explanation


### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

