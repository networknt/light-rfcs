### Summary
  This enhancement provides the ability to centrally manage configuration files. By 
  defining in a centralized management file called `values.json` or `values.yaml`
  or `values.yml`. default value located in different config file can be selectively 
  override with any value (Including environment variables by pattern). As the result,
  There would be no necessary to change previous code when config value need to be 
  changed.
  
  * Enable Flag

    Please define `true` or `false` for `enable_centralized_management` in commend line in 
    order to enable this enhancement. The default status is `true` without setting.

### Motivation


### Guide-level explanation
* Feature

  This enhancement support overriding Stirng, List, and Map.
  
  * Override String
  
    String can be overrided with any String, List or Map, including environment variables 
    by pattern.
    
  * Override List
  
    List can be overrided with any String, List, Map. It should be noticed that, the current 
    design is to replace the entire list if there is no nested Map which could be overrided
    ,since the value of the linked list cannot be located with key. Therefore, it does not 
    support adding new values to the original list. Maybe it will be supported later. 
    This List can be nested in Map. 
    
  * Override Map
  
    Map can be overrided partically. by provide partial key-value pair. The centralized
    management will nevigate to corresponding position of value and then override. This
    Map can be nested in List or Map. If the Map cannot be overrided with provided key-value, 
    whole map will be replaced.
  
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
      
### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
