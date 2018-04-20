### Summary

For all the synchronous request/response frameworks, we have a middleware handler to validate the request against specification during the runtime. It is hard to do the response validation in these frameworks as there is no better way to return the validation error to the user. 

After several discussion with core team members, we think the best location to do the response validation would be the client module. And this feature should be only used during the testing cycle and need to be turned off on production. 



### Motivation


### Guide-level explanation


### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

