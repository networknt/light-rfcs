### Summary
The `HealthGetHandler` is a convenient way to health check a light-4j service. It returns OK as the response without
"Content-Type" set in the response header. It does not have the message double quoted, therefore it can be deserialized
as Json String.
We need to add an option in its config, health.yml, to enable a Json response.

### Motivation
When a light4j service is deployed in an environment that is behind an ESB like "Data Power", its response message must be in a
unique format, otherwise, an error like "Invalid Json data format" will be returned from Data Power.
Since most of the handlers returns Json responses, we want the health check response to be in Json format as well.

### Guide-level explanation
In the config file of the `HealthGetHandler`, health.yml, add the following option:

```yaml
useJson: true
```
- When it is set to "true", the `HealthGetHandler` will return double quoted OK with "application/json" as the Content-Type
in the response header.
- When it is set to "false" or missing, it will return OK without double quoted and no Content-Type in the response header.
This is the same behavior as the current one to be backward compatible.

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
