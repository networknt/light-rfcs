### Summary
Currently, the config parameterization solution is backward incompatible and would cause the server to fail to start.

For example, `config.yml` should be excluded from parameter injection due to the introduce of new feature `DecrypterClass` in version 1.6.2. However, there are no corresponding updates in light-codegen. Therefore, when the `generateEnvVars` is enabled in light-codegen, the generated code would fail to execute due to loop invocation.

In conclusion, once some new config file introduced into our framework, we would need corresponding changes in light-codegen, which lead to the high couple between light-4j and light-codegen.  

### Motivation
Optimize the config parameterization to make it be backward compatible. 

### Guide-level explanation
1. Remove the handlerconfig and make templates out of them, this will allow us to handle them uniformly.

2. Parameterize the templates directly without using post-processed parameterization (overwrite the generated config). In this way, we can easily control which entry can be overwritten and which one cannot.

3. Provide a new codegen-cli commend `-p` for users to parameterize their own config file.
    
    For example,
    ```bash
    condegen-cli -f openapi -p examplePath/paramsCondig -o ...
    ```  

### Reference-level explanation


### Drawbacks
We need to ask the user to place the configuration files that need to be parameterized in a separate folder and strictly separate them from the framework configuration file to avoid the framework configuration file being overwritten twice. This may not be easy for users to manage their configuration file. 

### Rationale and Alternatives
Instead of doing `-p` stand alone, we can combine the hot reloading with config parameterization. When hot reloading is enabled, user can choose whether parameterize these hot reloading templates. In this way, people do not need to keep a separated folder for parameterized config.

And the commend can be simplified without specifying the directory of parameterization,
    
```bash
condegen-cli -f openapi -r -p ...
``` 

On the other word, let "config parameterization" be a function of hot overloading. Because of the configuration files in the framework we have completely controlled its composition with the template. Users only need to parameterize their custom configuration files, which are loaded by hot overload. So I think we should bundle these two features.

### Unresolved questions

