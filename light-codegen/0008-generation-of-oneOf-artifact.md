### Summary
The current Codegen is not able to handle code generation of oneOf artifact properly. It is handled by skipping the oneOf properties. We should find the corresponding generation method to make generation of oneOf meaningful.

### Motivation


### Guide-level explanation
One possible way is to generate all possible models corresponding to oneOf. For example,
```yaml
TestResult400:
    type: object
    allOf:
    - type: object
    properties:
        operationMetaData:
            $ref: '#/components/schemas/StandardResponse'
    - $ref: '#/components/schemas/ClientIdentifier'
ClientIdentifier:
    description: just some text
    type: object
    oneOf:
    - $ref: '#/components/schemas/TestID1'
    - $ref: '#/components/schemas/TestID2'
TestID1:
    type: object
    properties:
      id1:
        type: string
        maxLength: 15
TestID2:
    type: object
    properties:
      id2:
        description: test
        type: integer
        format: int64
```
There are two possibilities for TestResult400. It can contain the following two combinations of fields.
possible1:
```java
private StandardResponse operationMetaData;
private String id1;
```
possible2:
```java
private StandardResponse operationMetaData;
private long id2;
```

However, it seems that java has no way to represent multiple possibilities in the same class. So maybe we can generate multiple classes to represent. For example,
```
abstract class TestResult400 {
    private StandardResponse operationMetaData;
}
class TestID1Result400 extends TestResult400 {
    private String id1;
}
class TestID1Result400 extends TestResult400 {
    private long id2;
}
```
### Reference-level explanation


### Drawbacks
1. Model names can be complicated, especially when there are many levels of inheritance
2. Developers need to be familiar with inheritance relation to pick the right class

### Rationale and Alternatives


### Unresolved questions
