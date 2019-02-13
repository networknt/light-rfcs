# Status Enhancement

## Current behavior:

Whenever a project has its own defined status.yml, it overwrites the status.yml in light-4j. All the errors/info defined in light-4j are missing. To avoid this, each project is merging from status.yml in light-4j manually. It is inefficient and painful because whenever a new error code added in light-4j, they have to merge it manually again.

## A solution:

When load status.yml config, it loads both in light-4j and in its own project. We cannot do this based on current approach because when build the jar, the projects config files will overwrite the config file in each module. Thus when using

    public static final Map<String, Object> config = Config.getInstance().getJsonMapConfigNoCache(CONFIG_NAME);
    ------------------------------------------------------------------------------------------------------------
    inStream = getClass().getClassLoader().getResourceAsStream("config/" + configFilename);

it can only get either in its own project or in the framework.

I think we can to rename the status.yml in the framework to light-status.yml

and in Status.java load both status.yml and light-status.yml.

If there are conflicts(most likely in existing projects which are already merge with status.yml manually), the light-status.yml should overwrites the status.yml defined in its own project.(**need to discuss**)

## Backward Compatible:

I don't see any compatible issue unless they have duplicate error codes for their business with error codes for the platform. Besides those info in light-4j/status.yml is only for the framework.
