### Summary

This is a format guidance of values injection. Please reading the 0006-config-injection-guidance before 
reading this one if you do not familiar with how the config injection works.

### Motivation


### Guide-level explanation

1. To add a space after the colon inside curly bracket may be more readable. Therefore, we may need to support it.

Json config: 
```
    "key": "${something: default}"
    
    "key": "${something:default}"
```
Both above formats are acceptable. User can add or not add space after colon inside curly bracket.


Yaml config:
```
    key: ${something:default}
    
    key: "${something: default}"
```
Both above format are acceptable. User can add space after colon, but they need to use quotations to
include `${}` part.

2. `$` inside the curly bracket is the sign of skip, but if there are something after this sign, the sign not work.
```
    key: ${something:$} // skip the injection, and the output is key: ${something}
    
    key: ${something:$abc} // not skip, and the output is key: $abc
```
### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
