### Summary

When using light-codegen to generate POJO, only the schemas whose type is 
object can be generate properly. Otherwise, if the type is primitive types
or no type defined, the generation will be skipped.

Confusion may be caused by using the following contents in the specification
```
schemas:
Pet:
  required:
    - id
    - name
  properties:
    id:
      type: integer
      format: int64
    name:
      type: string
    tag:
      type: string
```
Since no type defined in this schema, the corresponding POJO cannot be generated.
However, it should be. 

### Motivation


### Guide-level explanation
The definition of the type should be mandatory, as it not only provides a more 
rigorous and reliable code generation specification, but can also be used in request 
verification and response verification.

Therefore, if the type is missing, an error contains the following message should be 
thrown.
```
Cannot find the type of "Account" in #/components/schemas/ of the specification file.
```

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions