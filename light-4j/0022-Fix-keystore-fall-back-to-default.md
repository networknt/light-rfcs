### Summary

User configures their HTTPS server keystore/trusttore but puts in the wrong filename, when the server doesn't find it, it will default to the open source test keystore/truststore and will run normally.

This is a security issue.

### Guide-level explanation

Perhaps the framework default test keystore and truststore should be excluded from the build. The way to achieve this is to migrate the test keystore and trustsore to the test folder. This way they will not be packaged.

However, when no keystore or truststore can be loaded. The server can still start normally. This is because if https is enabled, if the server does not have any corresponding keystore in any loaded path during the attempt to load the keystore, the server will create an empty keystore instance instead of throwing an exception. In this case, the client will not be able to get any response from the server. 

Maybe we should throw an exception in this case, so that the server can't start without keystore when https is enabled.

for example,

```java
if (configFilename.endsWith(CONFIG_EXT_KEYSTORE) || configFilename.endsWith(CONFIG_EXT_TRUSTSTORE)) {
  String message = "Unable to load '" + configFilename + ", please provide the keystore/truststore matching the configuration in client.yml/server.yml to enable ssl.";
  if (logger.isErrorEnabled()) {
      logger.error(message);
  }
  throw new RuntimeException(message);
}
```

### Drawbacks
N/A

### Rationale and Alternatives
N/A

### Unresolved questions
N/A
