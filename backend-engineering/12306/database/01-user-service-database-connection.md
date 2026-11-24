# User Service Database Connection

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [12306 overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Application Configuration](#1-application-configuration)
> - [2. ShardingSphere Configuration](#2-shardingsphere-configuration)

## 0. Overview

> The database connection configuration used by `user-service`.

- Service: `user-service`.
- Source paths below are relative to the `12306` project root.

1. [Application Configuration](#1-application-configuration): Loads the ShardingSphere configuration file.
2. [ShardingSphere Configuration](#2-shardingsphere-configuration): Configures database access through ShardingSphere.

## 1. Application Configuration

> Loads the ShardingSphere configuration file.

- File: `services/user-service/src/main/resources/application.yaml`.

```yaml
spring:
  datasource:
    driver-class-name: org.apache.shardingsphere.driver.ShardingSphereDriver
    url: jdbc:shardingsphere:classpath:shardingsphere-config.yaml
```

- `driver-class-name`: selects the ShardingSphere driver.
- `url`: loads the ShardingSphere configuration described below from the classpath.

## 2. ShardingSphere Configuration

> Configures database access through ShardingSphere.

- File: `services/user-service/src/main/resources/shardingsphere-config.yaml`.
- `dataSources`: MySQL connections.
  - `ds_0` → `12306_user_0`.
  - `ds_1` → `12306_user_1`.

  - JDBC URL breakdown (`ds_0`)

    ```text
    jdbc:mysql://HOST:PORT/DATABASE?OPTIONS
    ```

    | Part | Value in this example |
    | --- | --- |
    | JDBC prefix | `jdbc:mysql://` |
    | MySQL server address | `127.0.0.1` |
    | MySQL port | `3306` |
    | Database name | `12306_user_0` |
    | Connection options | Everything after `?`; options are separated by `&`. |

- `rules`: contains sharding and field encryption rules.
  - `!SHARDING`: routes SQL to the database and table.
    - `tables`: table layout and routing strategies.

      | Table name | `actualDataNodes` | `shardingColumn` | Database algorithm | Table algorithm |
      | --- | --- | --- | --- | --- |
      | `t_user` | `ds_${0..1}.t_user_${0..31}` | `username` | `user_database_hash_mod` | `user_table_hash_mod` |
      | `t_passenger` | `ds_${0..1}.t_passenger_${0..31}` | `username` | `passenger_database_hash_mod` | `passenger_table_hash_mod` |
      | `t_user_mail` | `ds_${0..1}.t_user_mail_${0..31}` | `mail` | `t_user_mail_database_hash_mod` | `t_user_mail_table_hash_mod` |
      | `t_user_phone` | `ds_${0..1}.t_user_phone_${0..31}` | `phone` | `t_user_phone_database_hash_mod` | `t_user_phone_table_hash_mod` |

      - `actualDataNodes` (`t_user` example): 2 databases × 32 table names = 64 declared combinations.
        - `ds_${0..1}` → data sources `ds_0`, `ds_1`.
        - `t_user_${0..31}` → table names `t_user_0`–`t_user_31` in each data source.

    - `shardingAlgorithms`: defines how the routing strategies calculate the destination database and table.
      - 4 sharded tables × 2 algorithms each = 8 entries.

      | Algorithm name | Routing level | `type` | `props` |
      | --- | --- | --- | --- |
      | `user_database_hash_mod` | Database | `CLASS_BASED` | `sharding-count: 32`<br>`table-sharding-count: 16`<br>`strategy: standard`<br>`algorithmClassName: org.opengoofy.index12306.framework.starter.database.algorithm.sharding.CustomDbHashModShardingAlgorithm` |
      | `user_table_hash_mod` | Table | `HASH_MOD` | `sharding-count: 32` |
      | `passenger_database_hash_mod` | Database | `CLASS_BASED` | `sharding-count: 32`<br>`table-sharding-count: 16`<br>`strategy: standard`<br>`algorithmClassName: org.opengoofy.index12306.framework.starter.database.algorithm.sharding.CustomDbHashModShardingAlgorithm` |
      | `passenger_table_hash_mod` | Table | `HASH_MOD` | `sharding-count: 32` |
      | `t_user_mail_database_hash_mod` | Database | `CLASS_BASED` | `sharding-count: 32`<br>`table-sharding-count: 16`<br>`strategy: standard`<br>`algorithmClassName: org.opengoofy.index12306.framework.starter.database.algorithm.sharding.CustomDbHashModShardingAlgorithm` |
      | `t_user_mail_table_hash_mod` | Table | `HASH_MOD` | `sharding-count: 32` |
      | `t_user_phone_database_hash_mod` | Database | `CLASS_BASED` | `sharding-count: 32`<br>`table-sharding-count: 16`<br>`strategy: standard`<br>`algorithmClassName: org.opengoofy.index12306.framework.starter.database.algorithm.sharding.CustomDbHashModShardingAlgorithm` |
      | `t_user_phone_table_hash_mod` | Table | `HASH_MOD` | `sharding-count: 32` |

      - Database calculation (`CLASS_BASED`): uses our custom Java class `CustomDbHashModShardingAlgorithm` to choose the database.

        ```text
        hash = Math.abs((long) shardKey.hashCode())
        databaseIndex = (hash % 32) / 16  // integer division
        ```

        - Result: index `0` → `ds_0`; index `1` → `ds_1`.
  - `!ENCRYPT`: encrypts configured fields.
    - `tables`: defines which table columns are encrypted.

      <table>
      <thead><tr><th>Table name</th><th>Column</th><th><code>cipherColumn</code></th><th><code>encryptorName</code></th></tr></thead>
      <tbody>
      <tr><td rowspan="4"><code>t_user</code></td><td><code>id_card</code></td><td><code>id_card</code></td><td><code>common_encryptor</code></td></tr>
      <tr><td><code>phone</code></td><td><code>phone</code></td><td><code>common_encryptor</code></td></tr>
      <tr><td><code>mail</code></td><td><code>mail</code></td><td><code>common_encryptor</code></td></tr>
      <tr><td><code>address</code></td><td><code>address</code></td><td><code>common_encryptor</code></td></tr>
      <tr><td rowspan="2"><code>t_passenger</code></td><td><code>id_card</code></td><td><code>id_card</code></td><td><code>common_encryptor</code></td></tr>
      <tr><td><code>phone</code></td><td><code>phone</code></td><td><code>common_encryptor</code></td></tr>
      </tbody>
      </table>

    - `encryptors`: defines the encryption algorithms used by those columns.

      | Encryptor name | `type` | `props` |
      | --- | --- | --- |
      | `common_encryptor` | `AES` | `aes-key-value` (encryption key) |

- `props`: runtime options, such as SQL logging.
