# Eventuate Event-Store Monitor

Create a UI - MicroService App/Tool for monitoring Eventuate/Tram Event-Store

It could be used for development environment debug or production support for the applications or services which use the following framework:

-- light-eventuate-4j
-- light-tram-4j
-- light-saga-4j

## Task list

### Backend

1. Service to get the event messages from Event-Store database;

2. Service to get the event messages from Event-Store message broker (Kafka);

3. Service to compare the result from task 1 and 2. It will verify cdc service process;

4. DashBoard service to search event/command/message by different search criteria from  Event-Store;

5. Monitor and report historic message in the Event-Store;

6. Monitor un-processed message in the Event-Store;


### Frontend

Create UI for the backend service, we prefer to create UI by react or angular2

