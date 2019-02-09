### Summary

The Config class offers very convenient methods to load config files in YML or JSON formats.

However, the methods are predicated on the files being available in a hard-coded directory,  which can be set with the environment variable "light-4j-config-dir", which is usually specified at startup with a "-D".

The current code is predicated on reading the config file from a directory relative to the "light-4j-config-dir"

### Motivation

There are use cases where a microservice or utility tool is required to read configs from either a relative or absolute path.

As a use-case, for example: the light-bot tool uses a relative path when merging statuses, and it is predicated on file being available under a common root

```
- project: abc
  file: status
  repository:
    - light-4j/status/src/main/resources/config
    - api-abc-status/src/main/resources/config/common
    - api-abc-status/src/main/resources/config/abcde
  output: api-abc/src/main/resources/config
  outputFormat: yml
```

With an absolute path, such code might be expressed, for example, as:

```
- project: abc
  file: status
  repository:
    - /Users/userX/work/light-4j/status/src/main/resources/config
    - /Users/userX/work/api-abc-status/src/main/resources/config/common
    - /Users/userX/work/api-abc-status/src/main/resources/config/abcde
  output: /Users/userX/work/api-abc/src/main/resources/config
  outputFormat: yml
```

### Guide-level explanation

The current code is limiting the loading to the "light-4j-config-dir" path, or paths relative to it:

```
  ...
  private InputStream getConfigStream(String configFilename) {
     InputStream inStream = null;
     try {
         inStream = new FileInputStream(EXTERNALIZED_PROPERTY_DIR + "/" + configFilename);
  ...
```

The implementation should check whether the path starts with a `/` and handle the situation of either:
- absolute path: `/Users/userX/work/light-4j/status/src/main/resources/config`
or
- relative path to "light-4j-config-dir": `light-4j/status/src/main/resources/config`

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives

The alternative is for clients to either fork the Config module and implement the functionality or build independent utility classes.

### Unresolved questions
