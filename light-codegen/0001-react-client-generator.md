### Summary

More and more Single Page Applications are being built with React or Angular and access backend APIs built on top of the light platform. We are constantly asked what is the best pattern for these applications. Should we put the JWT token to the localstorage in the browser or use light-router as BFF (backend for frontend) with server side session to keep the token there? In which case, should we have multiple light-router instances in the DMZ to provide high availability and high throughput? If multiple instances of light-router are used, how to enable distributed session? 

It depends on the use cases to answer above questions; however, we can abstract most typical scenarios and provide some patterns for most users to follow. Maybe just ask server questions then we can come up with something reasonable. 

Once we have serveral typcal patterns to follow, we can create client generator in the light-codegen. As there are numeric combinations of client side frameworks. We are going to pick React as the first one to work with. Other client side frameworks will follow later. 

On the server side, we are going to use OpenAPI 3.0 light-rest-4j implementation first as most people are using it currently. Other frameworks like light-graphql-4j and light-hybrid-4j can follow the similar pattern easily. 

### Motivation


### Guide-level explanation

For the client side React application, we are going to use the latest create-react-app to scaffolding an application and adding a menu system with all the endpoints access pages. These pages can be generated with react-schema-form based on the OpenAPI 3.0 schema or light-hybrid-4j schmem. For light-graphql-4j, we are going to explore relay or some other client side patterns. 

If users wants to use light-router as BFF, then we need to generate configuration files for it. Also, there would be a config item in config.json to control if distributed session is needed. If yes, then the distributed session handler will be wired in by default. 

To assist front end developers, it would be easy to setup a proxy in webpack for all the API calls. But on official test and prod enviroments, the front end SPA will talk to the backend services directly. 


### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

