### Summary

The light-portal was planned in the beginning when we start the light-4j framework as it is positioned to be the center of the infrastructure services which are part of the microservices ecosystem. 

When deploying a large scale of microservices, you need to have some crucial infrastructure services to support the life-cycle of the service from specification designed to runtime management to decommission the service. 

We have so many services scattered in different repositories, but we don't have a portal UI to integrate them together. I have started a UI in the [view](https://github.com/networknt/light-portal/tree/master/view) folder of light-portal, and there is a [tutorial](https://doc.networknt.com/tutorial/portal/view/) on how the UI is built and deployed. The next step is to integrate the [host-menu](https://github.com/networknt/light-portal/tree/master/host-menu) and [schema-form](https://github.com/networknt/light-portal/tree/master/schema-form) services to the light-portal UI. Once this is done, we can easily add new menu items and forms to extend it to other services. 

### Motivation

Customer demands drive the reason we need a short-term strategy for light-portal development.  We have a lot of customers ask if we can provide light-oauth2 services in the cloud so that they don't need to install them on premise. We also have users ask for the marketplace, config server and other services. 

The long-term strategy is to deploy SaaS(Software as a service) services on the cloud for small and medium-sized customers and package them as enterprise edition to large customers to install by themselves. 

We had a lot of services ready and deployed to the cloud already, but they are not easy to be consumed as we don't have a portal UI available. 

### Guide-level explanation

We need to complete the following tasks before moving to the next step of development. 

* Integrate the host-menu service
* Integrate the schema-form service
* Integrate the light-oauth2 services

Once we have the above services integrated to the light-portal, we will release it to the cloud test environment. Everyone can access the environment for free to support their development work. We will iron out small issues on the test environment and integrate the user-management to the light-portal at the same time. Once user-management is tested, we can start our production environment. 

### Reference-level explanation



### Drawbacks


### Rationale and Alternatives


### Unresolved questions

