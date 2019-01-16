### Summary

In current implementation, partial validators of path parameters and query 
parameters cannot work properly. 

The main reason for this is that the path parameters and the query parameters
are only recognized as strings since they are located in URL. 

In addition, URL decode is also needed before validating and arrays included by
query parameter should be validated entirely.
### Motivation


### Guide-level explanation
1. Fixed minimum validator and maximum validator

    If the parameters is in path or query, they can only be recognized as 
    strings. In this case, it should be judged to determine if it can be 
    converted to a number. then cast the parameter into double if possible
    to compared with minimum or maximum value set in specification.
    
    `typeLoose` is true if the parameter is in path or query.
    
    The code changed in `com.networknt.schema.MinimumValidator#validate`
    ```
    if (!node.isNumber() && !(config.isTypeLoose() 
                    && TypeFactory.getValueNodeType(node) == JsonType.STRING 
                    && TypeValidator.isNumeric(node.textValue()))) {
                // minimum only applies to numbers
                return Collections.emptySet();
            }
            String fieldType = this.getNodeFieldType();
    
            double value = node.asDouble();
    ```
    The code changed in `com.networknt.schema.MaximumValidator#validate`
    ```
    if (!node.isNumber() && !(config.isTypeLoose()
                    && TypeFactory.getValueNodeType(node) == JsonType.STRING
                    && TypeValidator.isNumeric(node.textValue()))) {
                // maximum only applies to numbers
                return Collections.emptySet();
            }
    
            String fieldType = this.getNodeFieldType();
    
            double value = node.asDouble();
    ```
2. Fixed pattern validator

    Since the path parameter used in path validation is directly intercepted
    from the URL which is not decoded, so when there is a special symbol, the 
    validator will not work properly.
    
    A decoding process is added before validating path parameter
    
    The code change in `com.networknt.openapi.RequestValidator#validatePathParameters`
    ```
    String decodeParamValue = null;
    
    try {
        decodeParamValue = URLDecoder.decode(requestPath.part(i), "UTF-8");
    } catch (Exception e) {
        logger.info("Path parameter cannot be decoded, it will be used directly");
    }

    final String paramValue = (decodeParamValue == null) ? requestPath.part(i) : decodeParamValue;
    ```
3. Fixed enum validator
    
    The reason why it cannot work properly is similar to minimum and maximum 
    validators. If the enum set in specification including numbers. these numbers
    cannot match with path parameters since all the parameters are strings.
    
    A private method called `isTypeLooseEqual` is created for this case.
    ```
    private boolean isTypeLooseEqual(JsonNode node) {
            if (config.isTypeLoose() && TypeFactory.getValueNodeType(node) == JsonType.STRING) {
                String nodeText = node.textValue();
                for (JsonNode n : nodes) {
                    String value = n.asText();
                    if (value != null && value.equals(nodeText)) {
                        return true;
                    }
                }
            }
            return false;
        }
    ```
    When the parameter is from path or query. this method will be invoke to 
    determine whether the parameter is included in enum.
    
    The code change in `com.networknt.schema.EnumValidator#validate`
    ```
    if (!nodes.contains(node) && !isTypeLooseEqual(node)) {
        errors.add(buildValidationMessage(at, error));
    }
    ```
    
4. Fixed array validation for query parameter validation 

    In current implementation, if query parameter is array, the validator can
    only do validate for each elements of it instead of validate the array entirely.
    Therefore, `ItemsValidator`,`MinItemsValidator` and `MaxItemValidator` which 
    is responsible for validating arrays and its length respectively would never
    be invoked in this case.
    
    Some codes are added in `com.networknt.openapi.RequestValidator#validateQueryParameter`
    The query parameter is an array if the size of `queryParameterValues` is larger than 1.
    For this parameter, the validator should be applied to it entirely.
    ```
    private Status validateQueryParameter(final HttpServerExchange exchange,
                                              final OpenApiOperation openApiOperation,
                                              final Parameter queryParameter) {
    
            final Collection<String> queryParameterValues = exchange.getQueryParameters().get(queryParameter.getName());
    
            if ((queryParameterValues == null || queryParameterValues.isEmpty())) {
                if(queryParameter.getRequired()) {
                    return new Status(VALIDATOR_REQUEST_PARAMETER_QUERY_MISSING, queryParameter.getName(), openApiOperation.getPathString().original());
                }
            } else if (queryParameterValues.size() < 2) {
    
                Optional<Status> optional = queryParameterValues
                        .stream()
                        .map((v) -> schemaValidator.validate(v, Overlay.toJson((SchemaImpl)queryParameter.getSchema())))
                        .filter(s -> s != null)
                        .findFirst();
                if(optional.isPresent()) {
                    return optional.get();
                }
            } else {
                Status status = schemaValidator.validate(queryParameterValues, Overlay.toJson((SchemaImpl)queryParameter.getSchema()));
                Optional<Status> optional = Optional.ofNullable(status);
                if(optional.isPresent()) {
                    return optional.get();
                }
            }
            return null;
        }
    ```
    
5. Summary of all validators for path parameters and query parameters which can
work properly

* Path parameter:
    
    i. Type schema can work properly for `integer`, `number`, `string`, `boolean`
    
    ii. If the schema type is `integer` or `number`, following schema can work properly
    depend on the specification.
    
    * minimum
    
    * maximum
    
    * minimumExclusive
    
    * maximumExclusive
        
    i.e.
    
    Specification
    ```
      {
        "in": "path",
        "name": "id",
        "description": "ID of Customer to retrieve.",
        "required": true,
        "schema": {
          "type": "integer",
          "minimum": 2,
          "maximum": 10,
          "exclusiveMinimum": true,
          "exclusiveMaximum": true,
        }
      }
    ```
    Valid request URL
    ```
    https://localhost:8453/v1/customers/5
    ```
    Invalid request URL
    ```
    https://localhost:8453/v1/customers/10
    ```   
    iii. If the schema type is `string`, the following schema should work properly
    
   *  MinLength
   *  MaxLength
       
    i.e.
    
    Specification
    ```
      {
        "in": "path",
        "name": "id",
        "description": "ID of Customer to retrieve.",
        "required": true,
        "schema": {
          "type": "string",
          "minLength": 2,
          "maxLength": 6
        }
      }
    ```
    Valid request URL
    ```
    https://localhost:8453/v1/customers/string
    ```
    Invalid request URL
    ```
    https://localhost:8453/v1/customers/stringverylong
    ```
    The enum schema can work properly when the type is `string`, `integer`, `number` and `boolean`
    
    i.e.
    
    Specification
    ```
      {
        "in": "path",
        "name": "id",
        "description": "ID of Customer to retrieve.",
        "required": true,
        "schema": {
          "type": "string",
          "enum": ["info_1", "info_2"]
        }
      }
    ```
    Valid request URL
    ```
    https://localhost:8453/v1/customers/info_1
    ```
    Invalid request URL
    ```
    https://localhost:8453/v1/customers/info_3
    ```
    
    The pattern schema can work properly
    
    i.e.
    
    Specification
    ```
      {
        "in": "path",
        "name": "id",
        "description": "ID of Customer to retrieve.",
        "required": true,
        "schema": {
          "type": "string",
          "pattern": "\\$\\{(.*?)\\}"
        }
      }
    ```
    Valid request URL
    ```
    https://localhost:8453/v1/customers/${123}
    ```
    Invalid request URL
    ```
    https://localhost:8453/v1/customers/123
    ```
    
    The following three schemas can also work properly
    
    * anyOf
    
    * oneOf
    
    * allOf
    
    i.e.
    
    Specification
    ```
      {
        "in": "path",
        "name": "id",
        "description": "ID of Customer to retrieve.",
        "required": true,
        "schema": {
          "oneOf": [
          {
            "type": "integer",
            "maximum": 200
          },
          {
            "type": "integer",
            "minimum": 500
          }]
        }
      }

    ```
    
    Valid request URL
    ```
    https://localhost:8453/v1/customers/150
    ```
    Invalid request URL
    ```
    https://localhost:8453/v1/customers/300
    ```
* Query parameter:
    
    All schema validators can work for path parameter can also work properly
    for query parameter.
    
    In addition, query parameter can support array validator now and following
    schema can work properly when schema type is `array`
    
    * minItems
    * maxItems
    
    i.e.
    Specification
    ```
      {
        "name": "idx",
        "in": "query",
        "description": "idx",
        "required": false,
        "schema": {
          "type": "array",
          "items": {
            "type": "string",
            "minLength": 2
          },
          "minItems": 2,
          "maxItems": 3
        }
      }

    ```
    
    Valid request URL
    ```
    https://localhost:8453/v1/customers/2?idx=12&idx=24
    ```
    Invalid request URL
    ```
    https://localhost:8453/v1/customers/2?idx=12&idx=24&idx=36&idx=48
    ```
    
    
### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

