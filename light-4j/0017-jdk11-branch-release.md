### Summary

The light platform 1.5.x is built on top of JDK 8 and it is production ready and we will support it until Java 8 is not supported. After JDK 11 is released, we have another jdk11 branch that contains the code on top of JDK 11. This branch will be the 2.0.X release, and any new feature and defect fix will be merged into two branches `develop` and `jdk11` at the same time. After the release for JDK8, the code will be merged from `develop` to `master`. For JDK11, the code will be merged from `jdk11` to `master-jdk11`. Once the majority of our customers moves to the JDK11, we are going to switch the develop and master branch to JDK 11 and JDK8 will become `jdk8` and `master-jdk8` for maintenance. In maintenance mode, only defect fixing will be back ported to the jdk8 branch.  

Now, for a period of time, we need to have two releases from different branches. That means we have to change our release process to automate it. 

### Motivation



### Guide-level explanation

The reason we need two branches for jdk11 and jdk8 is due to the difference in the codebase. For JDK11 migration, we have to make a lot of changes in the code and dependencies to ensure compilation in JDK 11. 

If we want to automate the release process, we need to update the light-bot to support flexibility. There are two options: enhance the light-bot or build light-bot-kotlin with DSL. I would prefer the second option as it is our long term strategy. 

For most of the changes, the same code can be merged into both `develop` and `jdk11` branches. Developers need to submit two separate PRs. If the code between jdk8 and jdk11 is different, then developers need to create two different feature branches and submit PRs to two branches. 

Before the development switch to JDK11, we encourage developers to keep the codebase the same in order to ensure that one fix can be merged to both branches. 

### Reference-level explanation


### Drawbacks

This increases the amount of work for developers as we have to think about both branches. Automation might take some workload from developers, but they need to have both JDK8 and JDK11 installed on the development machine. SDKMAN is a tool that is highly recommended. 

### Rationale and Alternatives



### Unresolved questions

