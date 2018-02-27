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
to install all the infrastructure services are road blocks for most developers. To
make developer's life easier, we have made decision to host all the depending services
in our cloud environment and share them with developers for free. We hope this can
attract more developers to build their services on our frameworks and more developers
to contribute to our platform. 

# Guide-level explanation


# Reference-level explanation

# Drawbacks

# Rationale and Alternatives

# Unresolved questions



