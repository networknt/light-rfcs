### Summary
If some server options become configurable, it may be possible to obtain flexible adaptations to the user's needs and extract max performance.

### Motivation


### Guide-level explanation
When setting these options, there are three cases:

1. All server options will be set to default values ​​when users do not provided server options.

2. When the server options are provided by user but these values are invalid, the default values are used and a warning is added in log.
 
    Invalid situations are as follows:
    * ioThreads <= 0
    * workerThreads <= 0
    * bufferSize <= 0
    * logback <= 0
    * serverString == null || serverString.equals("") 
    
3. Using the values provided by user when the server options are provided and the these server options are valid

### Reference-level explanation


### Drawbacks
These server configurations need to be carefully configured. We should emphasize careful configuration in the documentation and take risks at user's own risk.

This feature should only be used by advanced users.

### Rationale and Alternatives


### Unresolved questions
