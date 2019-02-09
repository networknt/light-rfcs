### Summary

The Config class offers very convenient methods to load config files in YML or JSON formats.

However, the methods are predicated on the files being available in a hard-coded directory,  which can be set with the environment variable "light-4j-config-dir", which is usually specified at startup with a "-D".

### Motivation

There are use cases where a microservice or utility tool is required to read configs from multiple directories or intends to segregate its configuration as such.      

This RFC wishes to propose an method which would allow the setting of the "light-4j-config-dir" as a list of directories, rather than a single directory.

As a use-case: a team wishes to segregate configurations in:
- configs for light-4j cross-cutting concerns
- configs for API-specific cross-cutting concerns

Alternatively, a team might wiosh to separate:
- configs for all middleware handlers
- API-specific business related config files

These are just examples.

### Guide-level explanation

The format of "light-4j-config-dir" could be, as an example: `-Dlight-4j-config-dir=/config:/etc/service:/var/secrets`


### Reference-level explanation


### Drawbacks


### Rationale and Alternatives



### Unresolved questions
