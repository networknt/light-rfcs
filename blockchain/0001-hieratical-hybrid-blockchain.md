### Summary

While working with light-eventuate-4j, light-tram-4j and light-saga-4j frameworks, we need to have an event store that can persist events immutablly. Currently, we are using Kafka as our message broker and MySQL database as our event store. There were a lot of discussions internally to evaluate Kafka itself as event store so that we can remove the dependency on MySQL; however, there are so many problems in operation when considering Kafka as event store. Also, we are facing some of the issues with Kafka as well. For example, it is depending on Zookeeper which is not actively maintained at all and log4j 1.2.17 jar has a security vulnerability. We have to approve that it is not a threat in our use case in order to make our customers comfortable to ignore this security scan result. 

The light platform is aiming fully distributed microservices implementations at enterprise level with Event Sourcing and CQRS support. And this is the same goal that the blockchain technologies are trying to resolve. In fact, all blockchain applications are using Event Sourcing to record transactions. Why can't we use a block chain as our event store. In that way we can build applications fully distributed on top of blockchain and take advantage of our knowledge on financial industry. 

In searching the suitable blockchains for our use case, we have evaluated several public chains, private chains and some open source projects. We found none of them meets our requirement. We need a fully distributed filesystem so that the blocks can be backed up by multiple nodes. We need high throughput so that it can handle millions transactions per second just like a Kakfa cluster.  


### Motivation

Given above findings, it might be a good idea to build a blockchain based on several existing technologies. For the filesystem, we can use IPFS. For the blocks, we can use similar technologies like Kafka that can easily scale with numeric topics. 

Also, I don't like the mining which burns a lot energy. We need a hieratical cluster so that end users can go to the right cluster to perform their daily transaction. 

### Guide-level explanation

* IPFS as filesystem to distribute blocks for backup.
* Kafka like system for blocks. There is a go lang implementation can be found [here](https://github.com/travisjeffery/jocko). 
* Hieratical clusters means you can have millions blocks working independently. 
* Hybrid cluster so that end user machines won't need to save all the data but only the blocks they are interested in. 
* Not all nodes are servers but most nodes are client only and only resposible for verifty the signature of the chains contains their own transactions. 
* Attract financial instituations and governments to form clusters of clusters. 
* Attract developers to build applications on the hybird chains. 

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

