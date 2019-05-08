### Summary

When a light-4j service requests to verify client JWT tokens, the framework needs to find out
how to resolve the key in the token from the Oauth server.

There are multiple ways to implement this resolver and the current way is to use the certificate
associated with the key.

We need to provide another way, which uses Json Web Keys as the key resolver. 
To be backward compatible, the current way will be treated as the default setting.
Once this is added to the framework, service providers are recommended to use this new way.

THe new resolver has already been provided in the library, `jose4j`. So the only change required in the framework
is to switch to the new resolver based on the setting in the `security` module.

### Motivation

Retrieving raw certificate from oauth server is not efficient
because the oauth server needs to use its REST management API interface to retrieve the cert from its truststore.

This API is not meant for this reason as in a management interface it requires the admin user name and password being provided in order to allow access.
This means that we have to hardcode somewhere the admin username and password which is a major management issue.

Using JSON Web Keys is a standard approach. A client (light-4j service) can query Oauth server
for the public key for a specific KID and the key will be used to validate the issued JWT token.

### Guide-level explanation

The `JwtHelper` will be able to use `JwksVerificationKeyResolver` as the new `VerificationKeyResolver`.

The certificate based resolver, `X509VerificationKeyResolver` will be kept for backward compatiblity reason.

Which one to be used will be controlled by the `security` module setting.

The `oauth.token.key` section in the `client` module setting must point to the JSON Web Keys service of the Oauth Server.

### Reference-level explanation

- security module

There is a new key, `keyResolver` added under `jwt`.
If the new key does not exists or set to `X509Certificate` or any other unknown values, the framework will still behave like before and uses the raw certificate to resolve the key.
If it is set to `JsonWebKeySet`, the framework will call the JSON Web Keys service on the oauth server.

```yaml
...
jwt:
  certificate:
    '100': primary.crt
    '101': secondary.crt
  clockSkewInSeconds: 60
  # This is the new key adde. Accepted values: JsonWebKeySet | X509Certificate
  keyResolver: X509Certificate
...
```

- client module

There is no change to the setting. Just make sure that the following setting point to the JSON Web Keys service.
The following is an example.

```yaml
...
oauth:
  ...
  token:
    ...
    key:
      # This must point to the JSON Web Keys service path
      uri: /oauth2/key
      # This is the service URL of the JSON Web Keys service
      server_url: https://cbvrhmccver01d.cloud1.cibc.com:8443
      # set to true if the oauth2 provider supports HTTP/2
      enableHttp2: false
...
```

### Drawbacks
N/A

### Rationale and Alternatives
N/A

### Unresolved questions
N/A
