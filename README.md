# druid

## 问题总结

### 最大空闲时间

默认情况下，mysql的连接超过8小时没有进行通信就会被mysql自动断掉，这个时间就是最大空闲时间

mysql断掉连接后，druid连接池会报如下异常：

```java
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: Communications link failure
The last packet successfully received from the server was 59,977 milliseconds ago.  The last packet sent successfully to the server was 1 milliseconds ago.
```

可以通过修改配置文件中的如下两个参数修改最大空闲时间：

```shell
mysql> show variables like '%timeout%'; 
+-----------------------------+----------+
| Variable_name               | Value    |
+-----------------------------+----------+
| connect_timeout             | 10       |
| delayed_insert_timeout      | 300      |
| have_statement_timeout      | YES      |
| innodb_flush_log_at_timeout | 1        |
| innodb_lock_wait_timeout    | 50       |
| innodb_rollback_on_timeout  | OFF      |
| interactive_timeout         | 28800    |			# 第1个
| lock_wait_timeout           | 31536000 |
| net_read_timeout            | 30       |
| net_write_timeout           | 60       |
| rpl_stop_slave_timeout      | 31536000 |
| slave_net_timeout           | 60       |
| wait_timeout                | 28800    |			# 第2个
+-----------------------------+----------+
13 rows in set (0.02 sec)
```

但是该参数没办法设置为不超时，所以这不是完美的办法，根本的办法是在druid端进行配置

```properties
# 检查连接使用的sql语句
spring.datasource.druid.validation-query=select 'x'
# 建议配置为true，不影响性能，并且保证安全性。申请连接时，如果空闲时间大于timeBetweenEvictionRunsMillis，执行validationQuery检测连接是否有效。
spring.datasource.druid.test-while-idle=true
# 申请连接时执行validationQuery检测连接是否有效，做了这个配置会降低性能
spring.datasource.druid.test-on-borrow=true
# 连接池判断连接是否超出最大空闲时间，一般设置为略小于数据库设置的最大空闲时间，单位毫秒，数据库中单位秒
spring.datasource.druid.time-between-eviction-runs-millis=20000000
```

