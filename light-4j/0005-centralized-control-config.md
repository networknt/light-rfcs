### Summary
This enhancement provides the ability to centrally manage configuration files. By 
defining in a centralized management file called `values.json` or `values.yaml`
or `values.yml`. default value located in different config file can be selectively 
override with any value (Including environment variables by pattern). As the result,
There would be no necessary to change previous code when config value need to be 
changed.

### Motivation


### Guide-level explanation
* Example

Suppose we need to change the port value and database password in the `server.yaml` 
and `service.yaml` respectively. Such `values.yaml` can be defined.
```
// values.yaml

server:
  port:8080

service:
  singleton:
    - javax.sql.DataSource:
      -com.zaxxer.hikari.HikariDataSource:
        password: mysql
```
```
// server.yaml
port: 4563
```
```
// service.yaml
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
  The result should be:
```
  // server.yaml
  port: 8080
```
```
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

### Reference-level explanation


### Drawbacks


### Rationale and Alternatives


### Unresolved questions
