### Summary

The light platform 1.5.x is built on top of JDK 8 and it is production ready and we will support it until Java 8 is not supported. After JDK 11 is release, we have another jdk11 branch that contains the code on top of JDK 11. This branch will be the 2.0.X release and any new feature and defect fixes will be merged to two branches develop and jdk11 at the same time. After the release for JDK8, the code will be merged from develop to master. For JDK11, the code will be merged from jdk11 to master-jdk11. Once majority of our customers moves to the JDK11, we are going to switch the develop and master branch to JDK 11 and JDK8 will become jdk8 and master-jdk8 for maintance. 

Now, for a period of time, we need to have two releases from to diferent branches. That means we have to change our release process to automate it. 

### Motivation


### Guide-level explanation

If we want to automate the release process, we need to update the light-bot to support the flexibility. There are two options: enhance the light-bot or build light-bot-kotlin with DSL. I would prefer the second option as it is our long term strategy. 

### Reference-level explanation


### Drawbacks

This increase the amount of the work for developers as we have to think about both branches. Automation might take some workload from developers but they need to have both JDK8 and JDK11 installed on the development machine. SDKMAN is the tool that is highly recommended. 


### Rationale and Alternatives



### Unresolved questions

1. Do we want to have one command line to release both jdk8 and jdk11? Or have two separate commands?
2. When do we switch our focus to jdk11? Waiting for Redhat docker image is ready?
3. How to merge one feature or fix to both develop and jdk11 branch? Two PRs?
4. Do we need to make sure that there is no significant changes in jdk11 in order to have one PR that can be merged to both branches?



