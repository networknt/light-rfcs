# Generate Test Cases Based on Specifications

## Validate Responses:

Right now we don't have validators to validate Response Schemas. For example:

    responses:
      '200':
        description: a todo item response
        content:
         'application/json':
            schema:
              type: object
              $ref: '#/components/schemas/TodoItem'
    	'400':
    	    description: some error
    	    content:
    	     'application/json':
    	        schema:
    	          type: object
    	          $ref: '#/components/schemas/ErrorInfo'

Should we add validate response schema feature in ValidatorHandler or in test cases?

- Do validating response in ValidatorHandler:

    Need to add a callback when Exchange complete get the response body, then validate.

        exchange.addExchangeCompleteListener((exchange, nextListener) -> {})

    By doing this way has some problems:

    We couldn't get response body efficiently, because undertow send response body through ConduitStreamSinkChannel directly, we could get response body by implementing Conduit and add to Response Wrapper. In this way, **no matter if the content passes the validation or not, the response content is already sent.** 

    **Doing validating response in ValidatorHandler is unrealistic**, unless we find another way to validate response content.

- Do response validation in test cases:

    We need to generate a valid request url based on request schemas, so that we can reach the handler we expect to test. Then, validate different scenarios based on status codes. For example:

        /todoItem/{itemId}:
        	put:
              description: update a todoItem
              operationId: updateTodoItem
              parameters:
              - name: itemId
                in: path
                required: true
                description: The id of the todo item to be updated
                schema:
                  type: string
        					pattern: (\w){2}-(\w){7}
              - name: param1
                in: query
                required: true
                description: The content of the todo item to be updated
                schema:
                  oneOf:
                    - $ref: '#/components/schemas/Schema2'
                    - $ref: '#/components/schemas/Schema3'

    The url should be /todoItem/nW-fiWcnCx?param1=Xdq7syKiDTRYrsf

    Thus, **a url generator** is needed to generate url based on schema to generate valid request url.

        ClientRequest request = new ClientRequest().setPath("/todoItem/nW-fiWcnCx?param1=Xdq7syKiDTRYrsf").setMethod(Methods.PUT);uestHeaders().put(Headers.TRANSFER_ENCODING, "chunked");
        connection.sendRequest(request, client.createClientCallback(reference, latch, "request body to be replaced"));

## About the url generator:

The url generator should support generate based on following types and properties in [Schema Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.0.0.md#schema-object):

integer or number:

minimum
maximum
minimumExclusive
maximumExclusive

string:

minLength

maxLength

pattern

enum:

string

integer

number

"[oneOf](https://swagger.io/docs/specification/data-models/oneof-anyof-allof-not/#oneof)", "[anyOf](https://swagger.io/docs/specification/data-models/oneof-anyof-allof-not/#anyof)" will also be supported, because we can pick the first schema as the schema.

"[allOf](https://swagger.io/docs/specification/data-models/oneof-anyof-allof-not/#allof)" won't be supported because it's unrealistic to find commons among all schemas.