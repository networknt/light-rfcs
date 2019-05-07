### Summary

In the previous release, @BalloonWen has submitted a [pull request](https://github.com/networknt/light-rest-4j/pull/69) to implement the response validator in the light-rest-4j. The implementation is just a class that can be invoked from the business handler manually. That means the developer must call the `ResponseValidator` before sending the response to the client with the following method. 

```
exchange.getResponseSender().send("response body in json");
```

After discussing with several users, we feel that it is not the optimal way to implement the response validation in the light platform. The major difference between light-4j and other frameworks is that we hide the cross-cutting concerns in the request/response chain as plugins so that developers can focus on the business logic only in the business handlers without thinking about how to wire in the cross-cutting concerns with annotations or method invocations. 

We also revisited the idea that allows the client module to validate the response and dropped the request as it is not feasible to have a workable solution in a short period. The main block is that our validator is depending on the specification and other handlers in the service request/response chain to catch the operations per endpoint(path/method). On the client side, there is no specification available. Even we can load the specification; we have to deal with multiple specs as one client might communicate with numerous services. 

### Motivation

It requires us to think further and find a solution to implement the response validator in the request/response chain like other middleware handlers. 

### Guide-level explanation

There are two options: 

* The normal response middleware handler. 

This is the normal way for Undertow to inject middleware handlers in the response chain by register a response listener in a middleware handler. We have several middleware handlers implemented this way already like metrics, audit, and dump, etc. They all need to capture information in the response to complete the middleware processing. The most closer middleware handler to the response validator is the dump handler which intercepts the response, deserializes the body, dumps it and serializes it again to send to the client. We can use the same way to do the validation, and send the body to the client. 

The drawback for this approach is that it is very slow as we are going through several serialization and deserialization cycles. That is why we don't recommend using the dump handler on production. We would say that the response validator shouldn't be used on production either. 

* Implement a ResponseSender wrapper

During the discussion with @BalloonWen, he proposed a new idea to implement the response cross-cutting concerns without sacrificing performance. The idea is the create a wrapper class to encapsulate the `exchange.getResponseSender().send` and ask all developers to invoke our wrapper class. Within the wrapper class, we can implement all response chain cross-cutting concerns that need to deal with the response body. I think even the metric response time and audit response status can be implemented in this way. 

The benefit is the performance as we are dealing with the body as a string first and then send to the Undertow channel. The drawback is that developers must call our wrapper instead of calling the Undertow method directly. For existing applications, a small change needs to be done to leverage the feature. 

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

We need the community to answer the following two questions: 

* Which above option should we implement? 
* When a validation error is encountered in the response chain, how to log the error? 
Currently, I am thinking to log the error in the log file and send the original body to the client. We do need to find a way to notify the developer if this kind of error occurs in the testing environment. We cannot send the error only as the client will be so confused without the original body. We also cannot combine the error with the original body as the result won't be valid again the specification. 
