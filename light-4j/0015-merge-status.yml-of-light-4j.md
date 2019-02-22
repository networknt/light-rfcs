# Status Enhancement

## Current behavior:

Whenever a project has its own defined status.yml, it overwrites the status.yml in light-4j. All the errors/info defined in light-4j are missing. To avoid this, each project is merging from status.yml in light-4j manually. It is inefficient and painful because whenever a new error code added in light-4j, they have to merge it manually again.

## Solution

Provide users with a configuration file called app-status.yml to place their own status. This file will be merged with the status.yml in the framework automatically. The benefits are:

**[1]** In the light-4j, almost every module has a config file that has the same name with the module. If we rename the framework's status.yml to light-status.yml, it would make some confusions.

**[2]** More backward compatible. For those people who prefer manually merge status.yml. There would be no changes for them. For those people who want to automatically merge, simply providing an app-status.yml. Their customized status codes would be merged automatically.

The merging rule should be:

**[1]** If the status code in app-status.yml is not contained by status.yml, the status code will be appended to default status list.

**[2]** If any status code in app-status.yml is contained by status.yml, a RuntimeException will be thrown and contains the list of duplicated status codes.

For example,
```
"The status codes: [ERR10000, ERR10001] in status.yml and app-status.yml are duplicated."
```

Since the status is lazy init, the merge process should be completed at startup to achieve the "fast fail". A method used to merge the status.yml and app-status.yml will be provided in the server module. And it will be added to the method init().
```
try {
// load config files from light-config-server if possible.
loadConfig();
// merge status.yml and app-status.yml if app-status.yml is provided
mergeStatusConfig();
start();
} catch (RuntimeException e) {
```
