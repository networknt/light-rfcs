### Summary

For networknt, we have specifications defined in the model-config repository so that light-codegen can quickly locate them to scaffold a new project. For enterprise-scale customers, they might have multiple organizations, and each organization has one or more repositories for specifications. To simplify the UI of light-portal, we need to introduce a service to load specifications from many locations and cache them for fast access. 

This API also handles the authorization for the specifications as it is not safe to handle it on the Single Page Application running on a browser. 

The service needs to be flexible, and all specification locations need to be part of the configuration file. 

### Motivation


### Guide-level explanation


### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

