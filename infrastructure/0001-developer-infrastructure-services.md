# Summary

This RFC proposes to install all infrastructure services in our dev cloud 
so that developers can use our shared services to jump start their projects
without installing them in their own data center or using docker-compose on
their laptops or desktops. 

The following services will be installed and exposed to public for testing
environment only. They are not supposed to be used for production. We will
setup production environment once we have sufficient demands from customers.

* light-oauth2

* Kafka/Zookeeper

* ELK

* InfluxDB/Prometheus/Grafana

* Consul

* Mysql

* ArangoDB

* light-portal(work in progress)

We will also deploy the some of the example applications to the environment
so that users can play with them. 


# Motivation

light platform can be used to build monolithic servics or microservices; however,
most people are using it to build microservices. The up front cost and complexity 
to install all the infrastructure services to support microservices architecture
are road blocks for most developers. To make developer's life easier, we have made 
decision to host all the depending services in our cloud environment and share them 
with developers for free. We hope this can attract more developers to build their 
services on our frameworks and more developers to contribute to our platform. 

# Guide-level explanation

All services will be installed in one of the VMs called api in the cloud and will 
be cleaned up everything month?

# Reference-level explanation

Once this environment is up and running we are going to update the light-codegen
to automatically point to it for the newly generated project. 

Also, we need to get the light-portal completed so that users can use portal to
gain more control for the dev envionment.



# Drawbacks

* Reponsibility

As this is a shared environment between different individuals and parties, there
might be some conflicts. We will be responsible to remove conflict or event rebuild
the environment when necessary. This might add a lot of workload to our team. 

* Security

Most services we want to expose are APIs and property security control must be in
place in order to make sure there are no spams or hacks. We still need to work out
a solution to control who can access Consul and Kafka etc.

* Availability

As this is an dev environment, we cannot guarantee it is up 24/7 like production. 
This might post challenges to developers who depending on it. We need to find a 
way to ensure high availability without too much more cost. 

# Rationale and Alternatives

Alternative solution would be for every developer to setup his/her environment on
cloud or their personal computer. This will not help developers' productivity. 

# Unresolved questions

* How to secure the environment? 

* How to grant the access to developers? 

* Are we going to use docker or just start these services as processes?

