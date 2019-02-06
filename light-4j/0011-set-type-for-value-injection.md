### Summary

This enhancement is the functional extension for values.yml. Please reading the 0006-config-injection-guidance before 
reading this one if you do not familiar with how the config injection works.

Currently, when we using the ${key:default_value} pattern to inject values. The type of the value is not correct sometimes. 
For example, the default value parsing from ${enable:true} is string "true", but it should be boolean. The reason is that
all the default value and values got from environment variables would be parsed as string.

To solve this problem, I decided to follow the snakeyaml's solution. That is, any environment variables and default value will be tried to be cast. all integers are converted to int, all decimals are converted to double, and all true and false are converted to Boolean values. 
In this way, we do not need to change anything in codegen and config files. Just some simple methods added to cast type. 

The benefits for doing this:
1. Make sure to get correct type of values
2. No need to change codegen and current config files

### Motivation


### Guide-level explanation
The only problem with this approach is that there are some corner problems. For example, people cannot set a string "true", 
"123" and such kinds of things. But these corner cases are rarely appeared. I think it is not worth to put effort there.

The following chart illustrate the type casting rule:

Notes: 
1. Only environment variables and default value will be cast, but the value from values.yml will keep its original type. Since values from values.yml have already been parsed by yml and no need to cast further. 
2. Strings that cannot be cast to int or double are treated as string.

| Integer | Double | Boolean | String |
| --- | --- | --- | --- |
| "1" | "1.0" | "true", "false" | "192.168.1.1" | 
| "+1" | ".1" | "True", "False" | "TurE" | 
| "-1" | "1." | "TRUE", "FALSE" | "abcd" | 


### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions




