### Summary
When generating models based on the openapi specification, an unexpected model 
class would be generated even though its schema isn't object. In addition, if 
the schema's type is object but no properties, an invalid class would also be 
generated.

The are two reasons caused this issue:

1. No type check when generate model
2. No properties number check in generation template

### Motivation


### Guide-level explanation
1. Added type check when generate model. Generation will be executed only if the 
type of the schema equals to `object`. Since generate a model for array, integer,
number, boolean does't make sense.

    Codes added in `com.networknt.codegen.rest.OpenApiGenerator#generate` 
    ```
    if (!"object".equals(type)) {
        continue;
    }
    ```
2. Added properties number check before equals() and hashCode() methods in 
`templates/rest/pojo.rocker.raw`. Since if a POJO does not have any fields, 
there is no way to determine if it is equals to other or generate a hash code.
    
    The modified parts show below
    ```
    @if(props.size() > 0) {
    
    @@Override
    public boolean equals(Object o) {
        if (this == o) {
            return true;
        }
        if (o == null || getClass() != o.getClass()) {
            return false;
        }
        @className @classVarName = (@className) o;

        return @for ((i, prop): props) {@if (i.index() <  props.size() - 1) {Objects.equals(@prop.get("name"), @classVarName.@prop.get("name")) &&}
        @if(i.index() == props.size() - 1){Objects.equals(@prop.get("name"), @classVarName.@prop.get("name"))}};
    }

    @@Override
    public int hashCode() {
        return Objects.hash(@for((i, prop): props) {@if(i.index() < props.size() - 1) {@prop.get("name"),} @if(i.index() == props.size() - 1) {@prop.get("name"));}}
    }

    }
    ```
3. Comparision
    
    Before modifying, if we defined specification like:
    ```
    schemas:
        accounts:
          type: array
          items:
            $ref: '#/components/schemas/account'
    ```
    A class called accounts.java would be generated and contains the following
    error codes:
    ```
        @Override
        public boolean equals(Object o) {
            if (this == o) {
                return true;
            }
            if (o == null || getClass() != o.getClass()) {
                return false;
            }
            Account account = (Account) o;
    
            return ;
        }
    
        @Override
        public int hashCode() {
            return Objects.hash(
        }
    ```
    There is nothing after `return` in equals() method and bracket is incomplete
    in hashCode() methods. The reason is that template try to get properties from
    specification but nothing can be found.
    
    After modifying, if the type of the schema isn't object. The model which is 
    corresponded to the schema would not be generated. otherwise, if the type is 
    object but no properties, an almost empty class would be generated like this:
    ```
    public class Accounts {
    
    
        public Accounts () {
        }
    
        @Override
        public String toString() {
            StringBuilder sb = new StringBuilder();
            sb.append("class Accounts {\n");
    
            sb.append("}");
            return sb.toString();
        }
    
        /**
         * Convert the given object to string with each line indented by 4 spaces
         * (except the first line).
         */
        private String toIndentedString(Object o) {
            if (o == null) {
                return "null";
            }
            return o.toString().replace("\n", "\n    ");
        }
    }
    ```
    This class doesn't seem to make any sense, but in order to prevent error when 
    other parts of the project depending on it, its generation is still necessary.
### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions