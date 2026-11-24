# Redis Integration

[All studies](../../README.md) · [Backend Engineering](../README.md) · [mall overview](README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Configuration](#1-configuration)
> - [2. Usage](#2-usage)

## 0. Overview

> Redis configuration and usage in mall.

1. [Configuration](#1-configuration)
   - Client settings
   - Server settings
2. [Usage](#2-usage)
   - Identity and authorization caching
     - Administrator accounts
     - Administrator resource lists
     - Member accounts
   - Temporary verification state
   - Atomic order-number sequencing
   - Native value types
     - String

## 1. Configuration

> Redis configuration has two sides: the applications configure their Redis clients, while the deployment configures the Redis server.

- Client settings
  - Application scope
    - `mall-common` defines the shared `RedisTemplate<String, Object>`, serializers, cache manager, and `RedisService` in `BaseRedisConfig`.
    - `mall-security` activates that shared client configuration through `RedisConfig`.
    - `mall-admin` and `mall-portal` use the Redis client.
    - `mall-search` explicitly excludes the transitive Redis starter and does not use Redis.
  - Connection
    - `mall-admin`
      - Development: `mall-admin/src/main/resources/application-dev.yml`
      - Production: `mall-admin/src/main/resources/application-prod.yml`
    - `mall-portal`
      - Development: `mall-portal/src/main/resources/application-dev.yml`
      - Production: `mall-portal/src/main/resources/application-prod.yml`
    - Values
      - Development host: `localhost`; production host: `redis`.
      - Port: `6379`; Redis database index: `0`; connection timeout: 300 milliseconds.
      - The checked-in configurations use no Redis password.
      - The separate `redis.database: mall` property is a key namespace, not the Redis database index.
  - Data handling
    - Keys and hash keys use string serialization.
    - Values use Jackson JSON serialization with Java type metadata.
    - `RedisService` exposes operations for String, Hash, Set, and List values, but the studied application code only calls String-value operations: `get`, `set`, `del`, and `incr`.
    - A one-day `RedisCacheManager` default is configured, but no Spring cache annotations use it at the studied revision.
- Server settings: `document/docker/docker-compose-env.yml`
  - Image: `redis:7`.
  - Container name: `redis`.
  - Startup command: `redis-server --appendonly yes`.
  - Data volume: `/mydata/redis/data:/data`.
  - Port mapping: `6379:6379`.

## 2. Usage

<table>
  <thead>
    <tr>
      <th>Module</th>
      <th>Key</th>
      <th>Value</th>
      <th>Purpose</th>
      <th>Expiry</th>
      <th>Used in</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2"><code>mall-admin</code></td>
      <td><code>mall:ums:admin:&lt;username&gt;</code></td>
      <td>Administrator account record<br><code>String · JSON object</code></td>
      <td>Avoids repeatedly loading the administrator record from MySQL during login and user-detail resolution.</td>
      <td>24 hours; invalidation runs after account updates, password changes, deletion, and logout, subject to the username lookup limitation described below.</td>
      <td><code>UmsAdminCacheServiceImpl</code>, <code>UmsAdminServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>mall:ums:resourceList:&lt;adminId&gt;</code></td>
      <td>Resources authorized for one administrator<br><code>String · JSON array</code></td>
      <td>Avoids repeatedly rebuilding the administrator's authorization resource list from role and resource relations in MySQL.</td>
      <td>24 hours; deleted after administrator-role changes, role-resource changes, resource updates or deletion, administrator deletion, or logout.</td>
      <td><code>UmsAdminCacheServiceImpl</code>, <code>UmsAdminServiceImpl</code>, <code>UmsRoleServiceImpl</code>, <code>UmsResourceServiceImpl</code></td>
    </tr>
    <tr>
      <td rowspan="3"><code>mall-portal</code></td>
      <td><code>mall:ums:member:&lt;username&gt;</code></td>
      <td>Member account record<br><code>String · JSON object</code></td>
      <td>Avoids repeatedly loading the member record from MySQL during login and user-detail resolution.</td>
      <td>24 hours; deleted after password or loyalty-point changes.</td>
      <td><code>UmsMemberCacheServiceImpl</code>, <code>UmsMemberServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>mall:ums:authCode:&lt;telephone&gt;</code></td>
      <td>Six-digit verification code<br><code>String</code></td>
      <td>Validates member registration and password-reset requests for a telephone number.</td>
      <td>90 seconds; not explicitly deleted after successful verification.</td>
      <td><code>UmsMemberCacheServiceImpl</code>, <code>UmsMemberServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>mall:oms:orderId&lt;yyyyMMdd&gt;</code></td>
      <td>Daily order sequence<br><code>String · integer</code></td>
      <td>Atomically allocates the sequence appended to the date, source type, and payment type when constructing an order number.</td>
      <td>No expiry is set; each date creates a new counter key.</td>
      <td><code>OmsPortalOrderServiceImpl</code></td>
    </tr>
  </tbody>
</table>
