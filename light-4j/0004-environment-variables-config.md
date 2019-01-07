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
    
  * Examples
  
    | Environment Variable | Input | Output | Action |
    | --- | --- | --- | --- |
    | VAR: default | value: ${VAR} | value: default | inject environment variable |
    | VAR: null | value: ${VAR} | throw ConfigException | exception |
    | VAR: null | value: ${VAR:default} | value: default | inject default |
    | VAR: default | value: ${VAR:$} | value: ${ENV} | skip injection | 
  
  * Enable Flag
  
    Please define `true` or `false` for `enable_centralized_management` in commend line in order to enable this enhancement.
  The default status is `true` without setting.

### Motivation


### Guide-level explanation
* Implementation
  
    When the config data is first loaded from the configuration file, a 
  preprocessor is used to preprocess the file's string based on the 
  replacement rules, and then use the resulting String for configuration 
  loading.
  ```
    String injectedConfigString = EnvInjection.inject(ConfigString);
  ```
    As shown above, the `EnvInjection` provides a method called `inject` 
    for preprocessing json and yaml respectively. The details are as follows
  
    1. Convert InputStream into String for following operation.
  
    2. Find the reference contents according to the regex pattern as follows
  ```
    private static Pattern pattern = Pattern.compile("\\$\\{(.*?)\\}");
  ```
      * `\\$\\{` and `\\}` means contain `${` and `}`
      * `(.*?)` means this part can match with every character sequneces
    
    3. process the reference contents`${(.*?)}` into the corresponding envirment 
    variable.
  ```
        private static EnvEntity getEnvEntity(String contents) {
        if (contents == null || contents.equals("")) {
            return null;
        }
        EnvEntity envEntity = new EnvEntity();
        contents = contents.trim();
        if (contents == null || contents.equals("")) {
            return null;
        }
        String[] array = contents.split(":", 2);
        if ("".equals(array[0])) {
            return null;
        }
        envEntity.setEnvName(array[0]);
        if (array.length == 2) {
            if (array[1].startsWith("?")) {
                envEntity.setErrorText(array[1].substring(1));
            }else if(array[1].startsWith("$")) {
                envEntity.setDefaultValue("\\$\\{" + array[0] + "\\}");
            }else {
                envEntity.setDefaultValue(array[1]);
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
  

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives
  We hope to further expand this pre-processing chain. Next we will try to implement a centralized configuration. See more details in https://github.com/jiachen1120/light-rfcs/edit/develop/light-4j/0004-environment-variables-config.md


### Unresolved questions
  Adding new line into curly bracket is not supported
  i.e. `${\n}`
