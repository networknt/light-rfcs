### Summary
  This enhancement provides the ability to centrally manage configuration files. By 
  defining in a centralized management file called `values.json` or `values.yaml`
  or `values.yml`. Values located in different config file can be selectively 
  override with any value (Including environment variables by pattern). As the result,
  There would be no necessary to change previous code when config value need to be 
  changed.
  
  * Enable Flag

    Please define `true` or `false` for `enable_centralized_management` in commend line in 
    order to enable this enhancement. The default status is `true` without setting.

### Motivation


### Guide-level explanation
* Feature

  This enhancement support overriding Stirng, List, and Map, regardless nested in List or Map.
  
  * Override String
  
    String can be overrided with any String, List or Map, including environment variables 
    by pattern.
    
  * Override List
  
    List can be overrided with any String, List or Map, including environment variables 
    by pattern.
    
  * Override Map
  
    Map can be overrided with any String, List or Map, including environment variables 
    by pattern.
  
* Example

  Suppose we need to change the port value and database password in the `server.yaml` 
  and `service.yaml` respectively. Such `values.yaml` can be defined.
  
  * values.yaml
  ```
    server:
      port:8080
    service:
      password: mysql

  ```
  
  * server.yaml
  ```
    port: 4563
  ```

  * service.yaml
  ```
    singletons:
    - javax.sql.DataSource:
      - com.zaxxer.hikari.HikariDataSource:
          DriverClassName: oracle.jdbc.pool.OracleDataSource
          jdbcUrl: jdbc:oracle:thin:@localhost:1521:XE
          username: SYSTEM
          password: oracle
          maximumPoolSize: 10
          useServerPrepStmts: true
          cachePrepStmts: true
          cacheCallableStmts: true
          prepStmtCacheSize: 10
          prepStmtCacheSqlLimit: 2048
          connectionTimeout: 2000
  ```
  
  * Result
  ```
    // server.yaml
    port: 8080
  
  
    // service.yaml
    singletons:
    - javax.sql.DataSource:
      - com.zaxxer.hikari.HikariDataSource:
          DriverClassName: oracle.jdbc.pool.OracleDataSource
          jdbcUrl: jdbc:oracle:thin:@localhost:1521:XE
          username: SYSTEM
          password: mysql
          maximumPoolSize: 10
          useServerPrepStmts: true
          cachePrepStmts: true
          cacheCallableStmts: true
          prepStmtCacheSize: 10
          prepStmtCacheSqlLimit: 2048
          connectionTimeout: 2000
  ```
  
  * Implementation
  
    1. The configuration file is first parsed into the map because it is easier to merge than using 
    the object.
    
    2. The value.yaml is also parsed into map.
    
    3. There are two methods provided by `CentralizedManagement.class` to merge the value of 
    values.yaml into ObjectConfig or MapConfig respectively.
    ```
    public static Map<String, Object> merge(String configName, Map<String, Object> config) {
        if (valueConfig == null) {
            logger.error("centralized management file \"values.yaml\" cannot be found");
            return config;
        }
        merge(config, valueConfig.get(configName));
        return config;
    }

    public static Object merge(String configName, Map<String, Object> config, Class clazz) {
        if (valueConfig == null) {
            logger.error("centralized management file \"values.yaml\" cannot be found");
            return convertMapToObj(config, clazz);
        }
        merge(config, valueConfig.get(configName));
        return convertMapToObj(config, clazz);
    }
    ```
    4. The merge process is recursive.
    ```
    private static void merge(Object m1, Object m2) {
        if (m1 instanceof Map && m2 instanceof Map) {
            Iterator<String> fieldNames = ((Map<String, Object>) m1).keySet().iterator();
            while (fieldNames.hasNext()) {
                String fieldName = fieldNames.next();
                Object field1 = ((Map<String, Object>) m1).get(fieldName);
                Object field2 = ((Map<String, Object>) m2).get(fieldName);
                if (field1 != null && field2 == null
                        && (field1 instanceof Map || field1 instanceof List)) {
                    merge(field1, m2);
                } else if (field2 != null) {
                    ((Map<String, Object>) m1).put(fieldName, field2);
                }
            }
        } else if (m1 instanceof List) {
            for (Object field1 : ((List<Object>) m1)) {
                merge(field1, m2);
            }
        }
    }
    ```
    
      
### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
