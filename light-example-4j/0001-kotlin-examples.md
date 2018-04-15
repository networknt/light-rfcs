### Summary

This RFC proposes to create several example applications in Kotlin as well as tutorials. 

The first one will be the pet store equivalent, and more examples should follow. 

### Motivation

The light platform is built for JVM and Kotlin can be seamlessly integrated with existing Java-based frameworks. To take advantages of the new language, we need to teach users on how to use it. Examples and tutorials will help users to grasp the technical details. By copying/pasting an example, they can start their projects quickly. 

Once we have several examples, we can abstract the pattern and create a generator in light-codegen for Kotlin. 

### Guide-level explanation

First, convert the OpenAPI 3.0 pet store app to Kotlin. 

### Reference-level explanation

There are a lot of examples on the Internet.

### Drawbacks

Currently, we don't have experience on Kotlin and someone needs to learn it while working on the example. The implementation might not be best in Kotlin and need to iterate several times to get it better. 

### Rationale and Alternatives

An alternative solution would be for us to implement some real services as a showcase for the framework users. The real application can be anything in the portal or other infrastructure services. A corresponding tutorial must be provided to explain the steps and implementation details. 

### Unresolved questions

