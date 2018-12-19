### Summary
  This enhancement allows environment variables to be injected into the 
  configuration file (Json/Yaml/Yml) based on the following rules:
  
  * ${VAR} 
  
    retrive the value named "VAR" from system property and replace "${VAR}"
    with the value. If the value cannot be found, a ConfigException will 
    be throwed.
  
  * ${VAR:default} 
  
    retrive the value named "VAR" from system property and replace "${VAR}"
    with the value. If the value cannot be found, "${VAR}" will be replaced
    with "default".
  
  * ${VAR:?error}
  
    retrive the value named "VAR" from system property and replace "${VAR}"
    with the value. If the value cannot be found, a ConfigException contains
    "error" will be throwed.
  
  * ${VAR:$}
  
    don't inject value.

### Motivation


### Guide-level explanation
* Implementation
  
    When the config data is first loaded from the configuration file, a 
  preprocessor is used to preprocess the file's inputStream based on the 
  replacement rules, and then use the resulting inputStream for 
  configuration loading.
  ```
    InputStream inStream = EnvConfig.resolveYaml(inStream);
  ```
    or 
  ```
    InputStream inStream = EnvConfig.resolveJson(inStream);
  ```
  
    As shown above, class provides two methods for preprocessing json and 
  yaml respectively. The details are as follows
  
    1. Convert InputStream into String for following operation.
  
    2. Find the reference contents according to the regex pattern as follows
  ```
    private static Pattern pattern = Pattern.compile("[^/]\\$\\{(.*?)\\}(\")?");
  ```
      * `[^/]` means do not contain the `/`
      * `\\$\\{` and `\\}` means contain `${` and `}`
      * `(.*?)` means this part can match with every character sequneces
      * `(\")?` means `"` is a selective character
    
    3. process the reference contents into the envirment variable
  ```
    private static EnvEntity getEnvEntity(String contents) {
        EnvEntity envEntity = new EnvEntity();
        contents = contents.trim();
        if (contents == null || contents.equals("")) {
            return null;
        }
        String[] rfcArray = contents.split(":", 2);
        if ("".equals(rfcArray[0])) {
            return null;
        }
        envEntity.setEnvName(rfcArray[0]);
        if (rfcArray.length == 2) {
            if (rfcArray[1].startsWith("?")) {
                envEntity.setErrorText(rfcArray[1].substring(1));
            } else {
                envEntity.setDefaultValue(rfcArray[1]);
            }
        }
        return envEntity;
    }
  ```
    iii.1. get environment variables from system property
  ```
    private static Object getEnvVariable(String envName) {
        return System.getenv(envName);
    }
  ```
    iii.2. wrap the information got from reference contents into an object
  called EnvEntity
  ```
    private static class EnvEntity {
        private String envName;
        private String defaultValue;
        private String errorText;

        public String getErrorText() {
            return errorText;
        }

        public void setErrorText(String errorTest) {
            this.errorText = errorTest;
        }

        public String getDefaultValue() {
            return defaultValue;
        }

        public void setDefaultValue(String defaultValue) {
            this.defaultValue = defaultValue;
        }

        public String getEnvName() {
            return envName;
        }

        public void setEnvName(String envName) {
            this.envName = envName;
        }
    }

    private static InputStream convertStringToStream(String string) {
        return new ByteArrayInputStream(string.getBytes());
    }
  ```
  
    4. replace `\\$\\{(.*?)\\}` with the environment variable 
  
    5. convert the result String into InputStream, then output it.
  
* Tips

  Whether to add quotes does not affect the actual function
  i.e. `"${VAR}"` equivalent to `${VAR}`
  
  However, adding new line into curly bracket is not supported
  i.e. `${\n}`
  
  A flag called `isEnableInjection` is provided to enable or disable this feature
  
* Examples
  
  | Env Variables | Input | Output | Action |
  | --- | --- | --- | --- |
  | ENV: default | value: ${ENV} | value: default | inject environment variable |
  |  | value: ${ENV} | throw ConfigException | exception |
  |  | value: ${ENV:default} | value: default | inject default |
  | ENV: default | value: ${ENV:$} | value: ${ENV} | skip injection | 

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
