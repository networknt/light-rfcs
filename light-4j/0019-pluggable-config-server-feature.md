### Summary
Make current config server functionality as pluggable features so that clients can inject their own implementation of config server e.g. implementation that uses Spring config server

### Motivation
To open up light-4j so that clients can use their own config server implementation if required. 

A new DefaultConfigLoader can be created and current config server related code can be moved there. Clients can mention this loader class (or their own class) in startup.yml file. And then Server will just locate, instantiate & call this DefaultConfigLoader. This way, clients can write their own ConfigLoader and override current implementation.

### Guide-level explanation
To use the new ConfigLoader, it should be added to startup.yml so that Server can locate & trigger its init() method. The startup.yml file will host all the static/parmanent key/value pairs which are not supposed to changed dynamically.  

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
