### Summary
Make current config module loading strategy as pluggable features so that clients can inject their own implementation of config loader

Notes: different from config server one. This RFC aims to enhance current config module to support multiple config loading ways.

### Motivation
To open up light-4j so that clients can use their own config loader implementation if required. 

For example, some users prefer to use a spring-like configuration that uses a single configuration file instead of multiple configuration files for each module. By providing this pluggable feature, users can provide their own config loader implementation and configure it in config.yml. This allows their projects to have their own configuration loading methods.

### Guide-level explanation
To use the new ConfigLoader for config module usage, it should be added to config.yml

For example, 

```yaml
exclusionConfigFileList:
  - openapi
  - values
  - status
  - test_exclusion

decryptorClass: com.networknt.decrypt.AESDecryptor

configLoaderClass: com.networknt.config.TestConfigLoader
``` 

User can define their own config loading method by implemented the interface
```java
package com.networknt.config;

import java.util.Map;

public interface ConfigLoader {

    Map<String, Object> loadMapConfig(String configName, String path);

    Map<String, Object> loadMapConfig(String configName);

    <T> Object loadObjectConfig(String configName, Class<T> clazz, String path);

    <T> Object loadObjectConfig(String configName, Class<T> clazz);
}
```

These override methods are called automatically when the configuration is loaded, so the user does not need to make any changes to the existing code, just modify config.yml and provide an implementation of the config loader.

Therefore this design is backward compatible.

More detailed implementation can refer to this branch:
https://github.com/networknt/light-4j/tree/feat/pluggable-config-loader

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
