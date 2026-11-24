# Redis Integration

[All studies](../../README.md) · [Backend Engineering](../README.md) · [shortlink overview](README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Configuration](#1-configuration)
> - [2. Usage](#2-usage)

## 0. Overview

> Redis configuration and usage in Shortlink.

1. [Configuration](#1-configuration)
   - Client settings
   - Server settings
2. [Usage](#2-usage)
   - Native value types: String, Hash, Set, Stream.
   - Redisson components: Bloom filters, locks, and an inactive delayed queue.

## 1. Configuration

1. Client settings
   - Connection settings

     | Setting | Repository configuration |
     | --- | --- |
     | Applications | `gateway`, `admin`, `project`, `aggregation` |
     | Files | `<module>/src/main/resources/application.yaml` |
     | Connection fields | Host, port, and password |
     | Address | `127.0.0.1:6379` in all four applications |
     | Topology | One configured server; no Cluster or Sentinel configuration |
     | Client APIs | `StringRedisTemplate`; Redisson 3.27.2 in admin and project |
     | Aggregate mode | Admin and project share the aggregation application's connection settings |

   - Application settings

     | Component | Setting |
     | --- | --- |
     | User traffic limit | Enabled in admin and aggregation; 20 requests per user per 1-second window |
     | Statistics Stream | `short-link:stats-stream` |
     | Consumer group | `short-link:stats-stream:only-group` |
     | Consumer | `stats-consumer`; one worker thread per application instance |
     | Consumption | Batch size 10; poll timeout 3 seconds; automatic acknowledgment enabled |
     | Username Bloom filter | 100 million expected insertions; 0.1% false-positive probability |
     | Group-ID Bloom filter | 200 million expected insertions; 0.1% false-positive probability |
     | Short-URL Bloom filter | 100 million expected insertions; 0.1% false-positive probability |

2. Server settings: Not defined in this repository.

## 2. Usage

<table>
  <thead>
    <tr><th>Module</th><th>Key</th><th>Value</th><th>Purpose</th><th>Expiry</th><th>Used in</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><code>admin</code><br><code>gateway</code></td>
      <td><code>short-link:login:&lt;username&gt;</code></td>
      <td>Token → user JSON<br><code>Hash</code></td>
      <td>Stores login sessions; gateway validates tokens.</td>
      <td>30 minutes; login renews it; logout deletes it.</td>
      <td><code>UserServiceImpl</code>, <code>TokenValidateGatewayFilterFactory</code></td>
    </tr>
    <tr>
      <td rowspan="5"><code>admin</code></td>
      <td><code>short-link:user-flow-risk-control:&lt;username&gt;</code></td>
      <td>Request count<br><code>String · integer</code></td>
      <td>Lua atomically counts requests for user traffic limits.</td>
      <td>1 second by configuration; set on the first increment.</td>
      <td><code>UserFlowRiskControlFilter</code>, <code>user_flow_risk_control.lua</code></td>
    </tr>
    <tr>
      <td><code>userRegisterCachePenetrationBloomFilter</code></td>
      <td>Username membership<br><code>Redisson RBloomFilter</code></td>
      <td>Screens username availability before registration.</td>
      <td>No expiry is set.</td>
      <td>Admin <code>RBloomFilterConfiguration</code>, <code>UserServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>gidRegisterCachePenetrationBloomFilter</code></td>
      <td>Group-ID membership<br><code>Redisson RBloomFilter</code></td>
      <td>Screens generated group IDs before insertion.</td>
      <td>No expiry is set.</td>
      <td>Admin <code>RBloomFilterConfiguration</code>, <code>GroupServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>short-link:lock_user-register:&lt;username&gt;</code></td>
      <td>Registration lock<br><code>Redisson RLock</code></td>
      <td>Serializes registration for the same username.</td>
      <td>No explicit lease; unlocked in <code>finally</code>.</td>
      <td><code>UserServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>short-link:lock_group-create:&lt;username&gt;</code></td>
      <td>Group-creation lock<br><code>Redisson RLock</code></td>
      <td>Serializes group creation for one user.</td>
      <td>No explicit lease; unlocked in <code>finally</code>.</td>
      <td><code>GroupServiceImpl</code></td>
    </tr>
    <tr>
      <td rowspan="11"><code>project</code></td>
      <td><code>short-link:goto:&lt;fullShortUrl&gt;</code></td>
      <td>Destination URL<br><code>String</code></td>
      <td>Resolves short links without a MySQL read.</td>
      <td>Time to link expiry; permanent links: 2,626,560,000 ms (~30.4 days). Invalidated by relevant edits or recycling.</td>
      <td><code>ShortLinkServiceImpl</code>, <code>RecycleBinServiceImpl</code>, <code>LinkUtil</code></td>
    </tr>
    <tr>
      <td><code>short-link:is-null:goto_&lt;fullShortUrl&gt;</code></td>
      <td><code>-</code><br><code>String</code></td>
      <td>Caches missing, disabled, deleted, or expired link results.</td>
      <td>30 minutes; cleared on recovery or when an expired link becomes valid again.</td>
      <td><code>ShortLinkServiceImpl</code>, <code>RecycleBinServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>short-link:stats:uv:&lt;fullShortUrl&gt;</code></td>
      <td>Visitor cookie IDs<br><code>Set</code></td>
      <td>Detects a first-seen visitor for this short URL.</td>
      <td>No Redis expiry is set; the visitor cookie lasts 30 days.</td>
      <td><code>ShortLinkServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>short-link:stats:uip:&lt;fullShortUrl&gt;</code></td>
      <td>Visitor IP addresses<br><code>Set</code></td>
      <td>Detects a first-seen IP for this short URL.</td>
      <td>No expiry is set.</td>
      <td><code>ShortLinkServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>short-link:stats-stream</code></td>
      <td><code>statsRecord</code> → visit JSON<br><code>Stream</code></td>
      <td>Queues visits for background statistics writes.</td>
      <td>No expiry is set; entries are deleted after successful processing.</td>
      <td><code>ShortLinkStatsSaveProducer</code>, <code>ShortLinkStatsSaveConsumer</code>, <code>RedisStreamConfiguration</code></td>
    </tr>
    <tr>
      <td><code>short-link:idempotent:&lt;messageId&gt;</code></td>
      <td><code>0</code> = processing; <code>1</code> = completed<br><code>String</code></td>
      <td>Detects messages already being processed or completed.</td>
      <td>2 minutes; reset on completion; deleted on processing errors.</td>
      <td><code>MessageQueueIdempotentHandler</code>, <code>ShortLinkStatsSaveConsumer</code></td>
    </tr>
    <tr>
      <td><code>shortUriCreateCachePenetrationBloomFilter</code></td>
      <td>Full short-URL membership<br><code>Redisson RBloomFilter</code></td>
      <td>Screens generated codes and rejects unknown redirect URLs.</td>
      <td>No expiry is set.</td>
      <td>Project <code>RBloomFilterConfiguration</code>, <code>ShortLinkServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>short-link:lock:create</code></td>
      <td>Creation lock<br><code>Redisson RLock</code></td>
      <td>Serializes the alternative create-with-lock API.</td>
      <td>No explicit lease; unlocked in <code>finally</code>.</td>
      <td><code>ShortLinkServiceImpl#createShortLinkByLock</code></td>
    </tr>
    <tr>
      <td><code>short-link:lock:goto:&lt;fullShortUrl&gt;</code></td>
      <td>Cache-rebuild lock<br><code>Redisson RLock</code></td>
      <td>Serializes MySQL lookup and cache refill for one URL.</td>
      <td>No explicit lease; unlocked in <code>finally</code>.</td>
      <td><code>ShortLinkServiceImpl#restoreUrl</code></td>
    </tr>
    <tr>
      <td><code>short-link:lock:update-gid:&lt;fullShortUrl&gt;</code></td>
      <td>Group-move coordination<br><code>Redisson RReadWriteLock</code></td>
      <td>Group changes take the write lock; statistics writes take a read lock.</td>
      <td>No explicit lease; unlocked in <code>finally</code>.</td>
      <td><code>ShortLinkServiceImpl#updateShortLink</code>, <code>ShortLinkStatsSaveConsumer</code></td>
    </tr>
    <tr>
      <td><code>short-link:delay-queue:stats</code></td>
      <td>Visit records<br><code>Redisson RDelayedQueue / RBlockingDeque</code></td>
      <td>Deprecated delay-queue code; producer unused and consumer startup disabled.</td>
      <td>5-second delay; no expiry is set.</td>
      <td><code>DelayShortLinkStatsProducer</code>, <code>DelayShortLinkStatsConsumer</code></td>
    </tr>
  </tbody>
</table>
