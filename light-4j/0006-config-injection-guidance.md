### Summary
  This enhancement allows both environment variables and variables from file to be 
  injected into the configuration file (Json/Yaml/Yml) based on the following rules:
  
  * ${VAR} 
  
    retrive the value named "VAR" from environment variables or centralized management 
    file "values.yaml" and replace "${VAR}" with the value. If the value cannot be found, a 
    ConfigException will be throwed.
  
  * ${VAR:default} 
  
    retrive the value named "VAR" from environment variables or centralized management 
    file "values.yaml" and replace "${VAR}" with the value. If the value cannot be found, 
    "${VAR}" will be replaced with "default".
  
  * ${VAR:?error}
  
    retrive the value named "VAR" from environment variables or centralized management 
    file "values.yaml" and replace "${VAR}" with the value. If the value cannot be found, 
    a ConfigException contains "error" will be throwed.
  
  * ${VAR:$}
  
    skip injection.
    
  * Examples
  
    | Environment Variable | values.yaml | Input | Output | Action |
    | --- | --- | --- | --- | --- |
    | VAR: default | N/A | value: ${VAR} | value: default | inject environment variable |
    | N/A | N/A | value: ${VAR} | throw ConfigException | exception |
    | N/A | N/A | value: ${VAR:default} | value: default | inject default |
    | VAR: default | N/A | value: ${VAR:$} | value: ${ENV} | skip injection | 
    | N/A | VAR:default | value: ${VAR} | value: default | inject value from values.yaml | 
  
  * Work mode
  
   There are three injection mode:
   1. Inject from "values.yaml" only. mode code: "0"
   2. Inject from system environment first, then inject from "values.yaml". mode code "1"
   3. Inject from "values.yaml" first, then override with the values in the system environment.
      mode code "2"

### Motivation


### Guide-level explanation

1. Define `0` or `1` or `2` for `injection_order` by using commend line or code in order to set the 
   work mode. The default status is `2` without setting.
  
   Set system property from command line (“-D” option)
   ```
   java -Dinjection_order="1" application_launcher_class
   ```
   Set system property from code using System.setProperty() method
   ```
   System.setProperty("injection_order", "1");
   ```

2. (optional) Set environment variables by using commend line before run the application.
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
  Adding new line into curly bracket is not supported
  i.e. `${\n}`
