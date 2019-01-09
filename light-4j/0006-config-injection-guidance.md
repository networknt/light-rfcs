### Summary
  This enhancement allows both environment variables and variables from the file called `values.yaml` 
  to be injected into the configuration file (JSON/YAML/Yml) based on the following rules:
  
  * ${VAR} 
  
    retrieve the value named "VAR" from environment variables or centralized management file "values.yaml" and replace "${VAR}" with the value. If the value cannot be found, a 
    ConfigException will be thrown.
  
  * ${VAR:default} 
  
    retrieve the value named "VAR" from environment variables or centralized management file "values.yaml" and replace "${VAR}" with the value. If the value cannot be found, 
    "${VAR}" will be replaced with "default".
  
  * ${VAR:?error}
  
    retrieve the value named "VAR" from environment variables or centralized management file "values.yaml" and replace "${VAR}" with the value. If the value cannot be found, 
    a ConfigException contains "error" will be thrown.
  
  * ${VAR:$}
  
    skip injection.
    
  * Examples
  
    1. Basic injection rules overview
    
    | Environment Variable | values.yaml | Input | Output | Action |
    | --- | --- | --- | --- | --- |
    | VAR: default | N/A | value: ${VAR} | value: default | inject environment variable |
    | N/A | N/A | value: ${VAR} | throw ConfigException | exception |
    | N/A | N/A | value: ${VAR:default} | value: default | inject default |
    | VAR: default | N/A | value: ${VAR:$} | value: ${ENV} | skip injection | 
    | N/A | VAR:default | value: ${VAR} | value: default | inject value from values.yaml | 
    
    
    2. A mocked up example
    
    server.yml:
    ```
    buildNumber: ${server.buildNumber:latest}
    ```
    values.yml:
    ```
    server.buildNumber: 123
    ```
    environment variables:
    ```
    server.buildNumber=456
    ```
    
    case1: Both value source exist
    
    If the order code equals to [0]: buildNumber: 123
    
    If the order code equals to [1]: buildNumber: 123
    
    If the order code equals to [2]: buildNumber: 456
    
    If the order code equals not set:buildNumber: 456 (default[2])

    case2: Only environment variable is provided
    
    If the order code equals to [0]: buildNumber: latest
    
    If the order code equals to [1]: buildNumber: 456
    
    If the order code equals to [2]: buildNumber: 456
    
    If the order code equals not set:buildNumber: 456 (default[2])

    case3: only values.yaml is provided
    
    If the order code equals to [0]: buildNumber: 123
    
    If the order code equals to [1]: buildNumber: 123
    
    If the order code equals to [2]: buildNumber: 123
    
    If the order code equals not set:buildNumber: 123 (default[2])

    case4:Neither environment variable or values.yaml is not provided
    
    If the order code equals to [0]: buildNumber: latest
    
    If the order code equals to [1]: buildNumber: latest
    
    If the order code equals to [2]: buildNumber: latest
    
    If the order code equals not set:buildNumber: latest (default[2])
    
    
    3. Use cases of injecting List/Map
    
    By injecting list/map, IP whitelist can be config easier.
    
    whitelists.yaml:
    ```
    paths:${IPS}
    ```
    values.yaml:
    ```
    IPS:
      - '127.0.0.1'
      - '10.10.*.*'
    ```
    result of whitelists.yaml:
    ```
    paths:
      - '127.0.0.1'
      - '10.10.*.*'
    ```
    
    
  * Work mode
  
    There are three injection modes:
   
    [0] Inject from "values.yaml" only.
   
    [1] Inject from system environment first, then overwrite by "values.yaml" if exist.
   
    [2] Inject from "values.yaml" first, then overwrite with the values in the system environment.
   
      
  * Support Types
  
    The values to be injected can be String, List, Map.

### Motivation


### Guide-level explanation

1. (optional) Define `0` or `1` or `2` for `injection_order` by using command line or code in order to set the 
   work mode. The default status is `2` without setting.
  
   Set system property from command line (“-D” option)
   ```
   java -Dinjection_order="1" application_launcher_class
   ```
   Set system property from code using System.setProperty() method
   ```
   System.setProperty("injection_order", "1");
   ```

2. (optional) Set environment variables by using command line before running the application.
   ```
   export key=value
   ```
3. (optional) Create centralized management file "value.yaml"
   ```
   key1: value1
   key2: value2
   ```
4. run your light-4j application

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
