# WIP: Troubleshooting Database Issues with Spring Boot

## Query Analysis

[Explain and analyse query](./sql-queries.md#analyse-queries)

## Transactions and Connections (Spring Boot)

### Log SQL Queries
```yaml
spring:
  jpa:
    show-sql: true
    properties.hibernate:
      format_sql: true
      use_sql_comments: true
      
logging:
  level:
    org.hibernate:
      SQL: DEBUG
```
### Log Values 
```yaml
logging:
  level:
    org.hibernate:
      SQL: DEBUG
      type.descriptor.sql: TRACE
      type.descriptor.sql.BasicBinder: TRACE
      orm.jdbc.bind: TRACE
```

### Transactions

```yaml
logging:
  level:
    org.springframework:
      transaction: trace
      orm.jpa.JpaTransactionManager: DEBUG
      jdbc.datasource.DataSourceTransactionManager: DEBUG
```

### Connections

```yaml
logging:
  level:
    org.hibernate:
      resource.jdbc.internal.LogicalConnectionManagedImpl: TRACE
      engine:
        jdbc.connections.internal: TRACE
        transaction.internal: DEBUG

    # HikariCP: Pool-Infos, Housekeeper, physische Open/Close-Events
    com.zaxxer.hikari: DEBUG

```

AWS RDS [available connections for aurora](https://docs.aws.amazon.com/AmazonRDS/latest/AuroraUserGuide/aurora-serverless-v2.setting-capacity.html#aurora-serverless-v2.max-connections)

Interesting Metrics:
```shell
# shows active database connections
hikaricp_connections_active{application="my-application"}

# waiting connections -> could be indicator for future timeouts
hikaricp_connections_pending{application="my-application"}

# max usage of connections
hikaricp_connections_usage_seconds_max{application="my-application"}

# number of timeouted connections
hikaricp_connections_timeout_total{application="my-application"}
```

Hikari Properties:
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 25 # number of connections the service allows itself
      connection-timeout: 5000 
      leak-detection-threshold: 2000 # threshold, above connections will be logged incl. trace

```

## AWS Database Insights
- activate Performance Insights to identify long-running or blocking queries

```terraform
# link: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_instance#performance_insights_enabled-1

resource "aws_db_instance" "default" {
  allocated_storage             = 10
  db_name                       = "mydb"
  engine                        = "mysql"
  engine_version                = "8.0"
  instance_class                = "db.t3.micro"
  manage_master_user_password   = true
  master_user_secret_kms_key_id = aws_kms_key.example.key_id
  username                      = "foo"
  parameter_group_name          = "default.mysql8.0"
  performance_insights_enabled  = true
}

```

## Other Traps
- Database connection erros can be caused not only by unavailable database connections, but also by not available threads.
  - check blocking threads
  - consider using virtual threads