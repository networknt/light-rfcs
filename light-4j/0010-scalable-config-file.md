### Summary
Currently, we have an exclusions.yml to indicate which files counld not be
injected variables. However, because of its specific name and single structure,
for instance, there is only one pair of key-values. Maybe we can put its content
in a configuration file called config.yml (On the other words, rename exclusions.yml
to config.yml). This file serves as a configuration extension platform. Whenever
a new simple configuration feature is introduced in the future, and it is not
worthwhile to create a separate configuration file for it. We can put it in config.yml.

### Motivation


### Guide-level explanation
config.yml
```
exclusionConfigFileList:
	- openapi
	- values
	- status
```
Notes:
1. Simply list the config file names without extensions(.json, .yaml, .yml),
since config module would try to load file name with different extensions to 
see which one can be loaded. Also, using different extensions but the same
file name to do different config is not recommended.
2. Specification and `status` cannot be changed
3. `values` is used to inject values, so it should not be injected
4. `config` is not necessary to be included in the list, since in order
to get the list we need parse config.yml first. Then, in order to parse
config.yml, we need to check whether the file name is config to prevent
dead loop. Thus, it is not necessary to be included int the list again.

Need to change:
1. Config module
2. Codegen

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions

