### Summary
  This enhancement provides users with an alternative way to config their services. There are two configuration ways, one is to assign values directly in the configuration file just like what we did before. The other is to set the value to an environment variable and use the '$' symbol in the yaml file to mark the values which need to be configurable. During the deployment process, the "preprocessor" will get the corresponding value for these variables marked by '$'symbol.

### Motivation
  Both ways have their own benefit in different scenarios. For instance, when some variables that need to be configured are generic variables, for example, the URL of the database. The user can choose auto-inject and set its value to an environment variable for centralized management.

### Guide-level explanation

* environment variable reference format:
  
  ${VAR}
  
  Where VAR is the name of the environment variable.

  Each variable reference is replaced at startup by the value of the environment variable. The replacement is case-sensitive   and occurs before the YAML file is parsed. References to undefined variables are replaced by empty strings unless you specify a default value or custom error text.

* To specify a default value, use:

  ${VAR:default_value}
  
  Where default_value is the value to use if the environment variable is undefined.

* To specify custom error text, use:

  ${VAR:?error_text}
  
  Where error_text is custom text that will be prepended to the error message if the environment variable cannot be expanded.

* If you need to use a literal ${ in your configuration file then you can write $${ to escape the expansion.

### Reference-level explanation
  An environment variable "preproccessor" is been added after the parsing process of Json or Yaml. It provides two methods, "InjectMapEnv()" and "injectObjectEnv()" which able to inject environment variable into map or object respectively.

### Drawbacks
* Extra Time is needed for traversal the entries of map or the fields of object.
* There would be different format for Json and Yaml. For Json, the format is "\"${VAR}\"", otherwise, Json cannot parse the content without quotation marks.

### Rationale and Alternatives


### Unresolved questions
