### Summary
Currently when a configuration issue or some other problem results in the service 
failing to register in consul, the log message outputted to the user is that the 
service failed to bind to whatever port. This causes confusion. 

### Motivation


### Guide-level explanation
In the bind method of the server module, all exceptions are handled as output the 
message "failed to bind to port xxxx.". However consul registration is also implemented
in this method. It means that although the port is bound to xxxx port successfully but 
failed registered to consul, the output information would still be "failed to bind to 
port xxxx.". So this confusion was caused because the exception of registration was not 
handled separately. 

And even though the user can get the correct exception information. There are still problems 
that need to be resolved. That is, when the registration fails, the server is still running. 
So should we stop the server at this moment and output the "server stops due to registration 
failure."? 

The modification is to catch the exception for bind process before registering the service and 
put the code of registration process insided into the try catch block separately. Then throw a 
RuntimeException and output the following information.
```
"Failed to register on Consul, server stopped."
```
By doing this, the server would stop and following information will be show:
```
Failed to register on Consul, server stopped.

Process finished with exit code 1
```

Code changed in bind() method of `com.network.server.Server`
```
static private boolean bind(HttpHandler handler, int port) {
    try {
        ... // bind process
    } catch (Exception e) {
        System.out.println("Failed to bind to port " + port);
        if (logger.isInfoEnabled())
            logger.info("Failed to bind to port " + port);
        return false;
    }
    // application level service registry. only be used without docker container.
    if (config.enableRegistry) {
        // assuming that registry is defined in service.json, otherwise won't start
        // server.
        try {
            ... // register process
            // handle the registration exception separately to eliminate confusion
        } catch (Exception e) {
            System.out.println("Failed to register on Consul, server stopped.");
            if (logger.isInfoEnabled())
                logger.info("Failed to register on Consul, server stopped.");
            throw new RuntimeException(e.getMessage());
        }
    }

    if (config.enableHttp) {
        System.out.println("Http Server started on ip:" + config.getIp() + " Port:" + port);
        if (logger.isInfoEnabled())
            logger.info("Http Server started on ip:" + config.getIp() + " Port:" + port);
    } else {
        System.out.println("Http port disabled.");
        if (logger.isInfoEnabled())
            logger.info("Http port disabled.");
    }
    if (config.enableHttps) {
        System.out.println("Https Server started on ip:" + config.getIp() + " Port:" + port);
        if (logger.isInfoEnabled())
            logger.info("Https Server started on ip:" + config.getIp() + " Port:" + port);
    } else {
        System.out.println("Https port disabled.");
        if (logger.isInfoEnabled())
            logger.info("Https port disabled.");
    }

    return true;

}
```

The branch related to this issue is https://github.com/networknt/light-4j/tree/fix/error-message-when-fail-registration
### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
