### Summary

The Config class offers very convenient methods to load config files in YML or JSON formats.

However, the methods are predicated on the files being available in a hard-coded directory,  which can be set with the environment variable "light-4j-config-dir", which is usually specified at startup with a "-D".

A separate RFC will address the need to load the YML files from either an absolute or a relative path.

### Motivation

There are use cases where a microservice or utility tool is required to read configs from multiple directories or intends to segregate its configuration as such.      

As a use-case: the light-bot tool already employs similar code, as it must read status.yml files from multiple sources in order to merge them.
- tools as the networknt:light-bot project for loading config files from multiple locations
  - project: abc
    file: status
    repository:
      - light-4j/status/src/main/resources/config
      - api-abc-status/src/main/resources/config/common
      - api-abc-status/src/main/resources/config/abcde
    output: api-abc/src/main/resources/config
    outputFormat: yml

Another use case: configurations, respectively microservice specific data is intended to be loaded from separate directories
- specifying configurations in multiple folders, in addition to the standard /src/main/resources/config

### Guide-level explanation

```
The Config.loadMapConfig(String fileName) method can be overloaded with a directory location as:
  Config.loadMapConfig(String fileName, String directoryName).
```

Implicitly, the getConfigStream(String configFilename) method must be overloaded with a directory location as:
```
  Config.getConfigStream(String configFilename, String directoryName).
```

The current code is limiting the loading to:
```
  ...
  private InputStream getConfigStream(String configFilename) {
     InputStream inStream = null;
     try {
         inStream = new FileInputStream(EXTERNALIZED_PROPERTY_DIR + "/" + configFilename);
  ...
```
### Reference-level explanation


### Drawbacks


### Rationale and Alternatives

The alternative is for clients to either fork the Config module and implement the functionality or build independent utility classes.

### Unresolved questions
