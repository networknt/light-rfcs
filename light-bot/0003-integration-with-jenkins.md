### Summary

Currently, light-bot is just an executor which takes a YAML configuration file for control. We don't have a controller service to monitor the github.com changes to trigger the build. It makes sense to integrate the current implementation with Jenkins so that we can leverage the control plane of the Jenkins. 

In the future, we are going to make the light-bot as a microservice, and the commands can be passed by the HTTP requests or events. 



### Motivation


### Guide-level explanation


### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

