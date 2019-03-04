There are several discussions within the team to support light-codegen to leverage customized templates and rendering classes so that each organization can generate something specific to their own frameworks built on top of light-4j frameworks.

It is very easy to inject rendering classes as we have the service module designed for this purpose. In order to support customized templates, we need to enable rocker hot reloading. 

By default, rocker hot reloading is not enabled in codegen-cli.
A command option "--reload" or "-r" need to be added to codegen-cli to indicate that rocker hot-reloading should be enabled.

For rocker hot-reloading to work properly, the config file `rocker-compiler.conf` and the re-compiled tempaltes must be present in the classpath .
If `rocker-compiler.conf` and `codegen-cli.jar` are in the same folder, the command below can be used to run the codegen-cli (`target/classes` is the default output folder of re-compiled templates).  

````
java -cp target/classes/:./codegen-cli.jar com.networknt.codegen.Cli -f ...
````

A shell script codegen.sh is provided to help users setting up config files and output folders.
