# Redis Integration

[All studies](../../README.md) · [Backend Engineering](../README.md) · [12306 overview](README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Configuration](#1-configuration)
> - [2. Usage](#2-usage)
>   - [2.1 Business Data Caching](#21-business-data-caching)
>   - [2.2 Shared Coordination State](#22-shared-coordination-state)
>   - [2.3 Cache Protection and Consistency](#23-cache-protection-and-consistency)

## 0. Overview

> Redis configuration and usage in 12306.

> Studied revision: `daf3a27`, matching the [repository overview](README.md). Source paths below are relative to the upstream repository root.

1. [Configuration](#1-configuration)
   - Client settings
   - Server settings
2. [Usage](#2-usage)
   - [Business data caching](#21-business-data-caching): Stored application data that avoids repeated database or payment-provider work.
   - [Shared coordination state](#22-shared-coordination-state): Stored state that coordinates work across application instances.
   - [Cache protection and consistency](#23-cache-protection-and-consistency): Application logic that controls cache misses and keeps cached data aligned with database changes.

## 1. Configuration

> Redis configuration has two sides: the applications configure their Redis clients, while the deployment configures the Redis server.

- Client settings
  - Application scope
    - `frameworks/cache` brings in Spring Data Redis and `redisson-spring-boot-starter` 3.21.3.
    - [`CacheAutoConfiguration`][cache-config] exposes `StringRedisTemplateProxy` as the shared `DistributedCache`. Application code also obtains the underlying `StringRedisTemplate` for Hash, Set, pipelining, and Lua operations.
    - The four business services use Redis. In aggregate mode, `aggregation-service` hosts their code and supplies one shared configuration.
    - `gateway-service` declares no Redis connection configuration.
    - The Redisson starter can provide the `RedisConnectionFactory` used by `StringRedisTemplate`; the template is an API abstraction, not evidence of a separately configured Lettuce client. [Redisson 3.21.3 auto-configuration](https://github.com/redisson/redisson/blob/redisson-3.21.3/redisson-spring-boot-starter/src/main/java/org/redisson/spring/starter/RedissonAutoConfiguration.java).
  - Connection
    - Files: `services/<application>/src/main/resources/application.yaml`, for `user-service`, `ticket-service`, `order-service`, `pay-service`, and `aggregation-service`.
    - All five declare `spring.data.redis.host: 127.0.0.1` and `port: 6379`, with a development password.
    - They do not explicitly set the Redis database index, connection timeout, pool sizes, or TLS. No Cluster or Sentinel connection settings are checked in.
  - Value expiry
    - [`RedisDistributedProperties`][redis-properties] binds `framework.cache.redis`. Its base default is 30,000 milliseconds; the applications override it:

      | Application | Configured wrapper default |
      | --- | --- |
      | `user-service`, `aggregation-service` | `value-timeout: 50000`; inherited millisecond unit → 50 seconds |
      | `ticket-service`, `order-service`, `pay-service` | `value-timeout: 16`, `value-time-unit: days` → 16 days |

    - These settings control expiry in the shared cache wrapper. They are not connection timeouts or Redis server settings.
    - Calls with an explicit `TimeUnit` use their own expiry. Direct Hash and Set writes do not inherit the wrapper default.
  - Data handling and naming
    - `StringRedisTemplate` stores string keys and string values. [`StringRedisTemplateProxy`][cache-proxy] serializes application objects to Fastjson2 JSON.
    - `RedisKeySerializer` prepends `framework.cache.redis.prefix` to template keys; its default is empty, and the checked-in YAML does not override it.
    - Direct `RedissonClient` object names do not pass through that serializer. Selected lock and Bloom-filter names separately resolve `${unique-name:}`; this is not a universal Redis namespace.
  - Bloom-filter settings
    - [`UserRegisterBloomFilterProperties`][bloom-config] binds `framework.cache.redis.bloom-filter.user-register`.
    - `user-service` and `aggregation-service` name the filter `user_register_cache_penetration_bloom_filter${unique-name:}`.
    - Expected insertions default to `64`; false-positive probability defaults to `0.03`. These are small example settings, not demonstrated production sizing.
    - The separate generic filter in `CacheAutoConfiguration` requires `framework.cache.redis.bloom-filter.default.enabled=true`; the checked-in YAML does not enable it.
- Server settings
  - The repository supplies no Redis `redis.conf`, Docker Compose service, pinned server image, or server startup command. Redis must be provisioned separately.
  - Persistence, memory limits, eviction policy, replication, and server-side authentication are not specified. The client password does not establish how server authentication was provisioned.
  - There is no checked-in Redis Cluster or Sentinel deployment.
  - The MQ idempotency script executes `SET ... NX GET PX ...`. This combination requires Redis 7.0 or later, although the repository does not pin a server version. [Redis SET command history](https://redis.io/docs/latest/commands/set/#history).

## 2. Usage

Redis has three roles in this repository:

1. [Business data caching](#21-business-data-caching): Stored application data that avoids repeated database or payment-provider work.
2. [Shared coordination state](#22-shared-coordination-state): Stored state that coordinates work across application instances.
3. [Cache protection and consistency](#23-cache-protection-and-consistency): Application logic that controls cache misses and keeps cached data aligned with database changes.

### 2.1 Business Data Caching

> Stored application data that avoids repeated database or payment-provider work.

- Keys below use the default empty template prefix.
- `ADVANCE_TICKET_DAY` is `15`, so explicit uses of that constant expire after 15 days even where the wrapper default is 16 days.

<table>
  <thead>
    <tr><th>Module</th><th>Key</th><th>Value</th><th>Purpose</th><th>Expiry</th><th>Used in</th></tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2"><code>user-service</code></td>
      <td><code>&lt;accessToken&gt;</code></td>
      <td>Login response<br><code>String · JSON object</code></td>
      <td>Returns the saved login state through <code>checkLogin</code>.</td>
      <td>30 minutes; deleted on logout or account deletion.</td>
      <td><code>UserLoginServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>index12306-user-service:user-passenger-list:&lt;username&gt;</code></td>
      <td>Passenger records<br><code>String · JSON array</code></td>
      <td>Avoids querying the passenger list repeatedly, including lookup by passenger IDs.</td>
      <td>1 day; deleted after passenger creation, update, or removal. Empty lists are returned as null and are not cached.</td>
      <td><code>PassengerServiceImpl</code></td>
    </tr>
    <tr>
      <td rowspan="10"><code>ticket-service</code></td>
      <td><code>index12306-ticket-service:train_info:&lt;trainId&gt;</code></td>
      <td>Train record<br><code>String · JSON object</code></td>
      <td>Reuses train metadata during queries, purchase validation, and token-bucket initialization.</td>
      <td>15 days.</td>
      <td><code>TicketServiceImpl</code>, purchase filters, <code>TicketAvailabilityTokenBucket</code>, <code>SeatMarginCacheLoader</code></td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:region-station:&lt;nameOrQueryType&gt;</code><br><code>index12306-ticket-service:all_station</code></td>
      <td>Region or station lists<br><code>String · JSON array</code></td>
      <td>Serves station search, grouped region selection, and the full station list.</td>
      <td>15 days.</td>
      <td><code>RegionStationImpl</code></td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:query_all_region_list</code></td>
      <td>Region/station code → name<br><code>Hash</code></td>
      <td>Checks whether requested origin and destination codes exist before querying tickets.</td>
      <td>No expiry is set by the loader.</td>
      <td><code>TrainTicketQueryParamVerifyChainFilter</code></td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:region_train_station_mapping</code></td>
      <td>Station code → region name<br><code>Hash</code></td>
      <td>Resolves station codes to the region names used in ticket-list keys.</td>
      <td>No expiry is set by the V1 loader.</td>
      <td><code>TicketServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:region_train_station:&lt;fromRegion&gt;_&lt;toRegion&gt;</code></td>
      <td>Train/route identifier → ticket-list DTO JSON<br><code>Hash</code></td>
      <td>Stores the train list for a region pair. The active query key contains no travel date.</td>
      <td>No expiry is set by the V1 loader.</td>
      <td><code>TicketServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:train_station_price:&lt;trainId&gt;_&lt;departure&gt;_&lt;arrival&gt;</code></td>
      <td>Prices by seat type<br><code>String · JSON array</code></td>
      <td>Avoids repeated route-price queries.</td>
      <td>15 days.</td>
      <td><code>TicketServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:train_carriage:&lt;trainId&gt;</code></td>
      <td>Carriage type → comma-separated carriage numbers<br><code>Hash</code></td>
      <td>Finds carriages of the requested type for seat selection.</td>
      <td>No expiry is set by the loader.</td>
      <td><code>CarriageServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:train_station_stopover_detail:&lt;trainId&gt;</code></td>
      <td>Train stop records<br><code>String · JSON array</code></td>
      <td>Validates the requested journey against the train's route.</td>
      <td>15 days; populated on demand or by the stopover preload job.</td>
      <td><code>TrainPurchaseTicketParamVerifyChainHandler</code>, <code>TrainStationJobHandler</code></td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:train_station_remaining_ticket:&lt;trainId&gt;_&lt;departure&gt;_&lt;arrival&gt;</code></td>
      <td>Seat type → remaining count<br><code>Hash · integer fields</code></td>
      <td>Displays remaining tickets and performs preliminary stock checks. Purchase/cancellation paths change counts with <code>HINCRBY</code>.</td>
      <td>Lazy loading sets no expiry; the demo preload job sets 15 days. Incremental updates do not assign a new expiry.</td>
      <td><code>SeatMarginCacheLoader</code>, <code>TicketServiceImpl</code>, purchase handlers, <code>DelayCloseOrderConsumer</code>, Canal handler</td>
    </tr>
    <tr>
      <td><code>index12306-ticket-service:train_station_carriage_remaining_ticket:&lt;trainId&gt;_&lt;departure&gt;_&lt;arrival&gt;</code></td>
      <td>Carriage number → remaining count<br><code>Hash · integer fields</code></td>
      <td>Reads per-carriage counts if present; otherwise queries MySQL.</td>
      <td>No checked-in writer or expiry assignment was found.</td>
      <td><code>SeatServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>pay-service</code></td>
      <td><code>index12306-pay-service:order_pay_result:&lt;orderSn&gt;</code></td>
      <td>Payment-initiation response<br><code>String · JSON object</code></td>
      <td>Reuses the provider response when payment creation is requested again. This is not a final payment-status cache.</td>
      <td>10 minutes; no explicit deletion after successful payment was found.</td>
      <td><code>PayServiceImpl</code></td>
    </tr>
  </tbody>
</table>

1. Query versions
   - [`TicketServiceImpl`][ticket] V1 builds missing train-list and price caches and loads missing stock through [`SeatMarginCacheLoader`][seat-margin].
   - V2 pipelines price and remaining-stock reads to reduce network round trips. It expects those caches to exist; it does not reproduce V1's miss-rebuild logic.
2. Separate preload and legacy code
   - [`RegionTrainStationJobHandler`][region-job] writes a date-specific Sorted Set ordered by departure timestamp, with a 15-day expiry. It concatenates the unformatted `REGION_TRAIN_STATION` format string, leaving literal `%s_%s` in its key. It does not populate the region-pair Hash consumed by V1/V2 queries.
   - [`TrainStationRemainingTicketJobHandler`][remaining-job] writes hard-coded demonstration stock counts. The database-backed lazy loader is a separate path.
   - `TrainStationDetailJobHandler` is deprecated and writes separate 15-day detail Hashes.
   - Bitmap helpers contain `GETBIT` reads, but no current purchase-flow call to `checkSeat` or bitmap writer was found. They do not establish an active Redis bitmap inventory model.

### 2.2 Shared Coordination State

> Stored state that coordinates work across application instances.

<table>
  <thead>
    <tr><th>Module</th><th>Key or object name</th><th>Value</th><th>Purpose</th><th>Expiry</th><th>Used in</th></tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2"><code>user-service</code></td>
      <td><code>user_register_cache_penetration_bloom_filter&lt;unique-name&gt;</code></td>
      <td>Redisson <code>RBloomFilter&lt;String&gt;</code></td>
      <td>Tracks previously registered usernames for availability checks without querying MySQL for every proposed name.</td>
      <td>No application expiry is set; registration adds names, and account deletion leaves Bloom entries in place.</td>
      <td><code>RBloomFilterConfiguration</code>, <code>UserLoginServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>index12306-user-service:user-reuse:&lt;shard&gt;</code></td>
      <td>Reusable usernames<br><code>Set</code></td>
      <td>Allows a deleted username to be registered again despite its retained Bloom-filter entry.</td>
      <td>No expiry is set; deletion adds the username, and registration removes it.</td>
      <td><code>UserLoginServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>ticket-service</code></td>
      <td><code>index12306-ticket-service:ticket_availability_token_bucket:&lt;trainId&gt;</code></td>
      <td>Departure/arrival/seat-type → reservable count<br><code>Hash · integer fields</code></td>
      <td>Rejects V2 purchases with insufficient inventory before seat selection. Lua checks requested seat-type quantities and decrements affected route fields atomically.</td>
      <td>No expiry is set; cancellation or delayed closure returns tokens. Explicit deletion allows rebuilding from MySQL.</td>
      <td><code>TicketAvailabilityTokenBucket</code>, <code>TicketServiceImpl</code>, close-order handlers</td>
    </tr>
    <tr>
      <td><code>order-service</code><br><code>pay-service</code></td>
      <td><code>distributed_id_generator_config</code><br><code>distributed_pay_id_generator_config</code></td>
      <td>Startup node counters<br><code>String · integer</code></td>
      <td>Allocates generator node values using <code>INCR</code> under a distributed lock. IDs are subsequently generated in memory.</td>
      <td>No expiry is set; each counter resets to zero after exceeding 32.</td>
      <td><code>OrderIdGeneratorManager</code>, <code>PayIdGeneratorManager</code></td>
    </tr>
    <tr>
      <td><code>frameworks/distributedid</code></td>
      <td><code>snowflake_work_id_key</code></td>
      <td><code>dataCenterId</code> and <code>workId</code><br><code>Hash · integer fields</code></td>
      <td>Allocates a Snowflake worker pair through Lua at initialization when a Redis host is configured. Redis failure falls back to random selection.</td>
      <td>No expiry is set; values cycle within the script's bounded range.</td>
      <td><code>LocalRedisWorkIdChoose</code>, <code>chooseWorkIdLua.lua</code></td>
    </tr>
    <tr>
      <td><code>frameworks/idempotent</code></td>
      <td><code>&lt;annotation-prefix&gt;&lt;message-identity&gt;</code><br>Examples: <code>index12306-ticket:delay_close_order:</code>, <code>index12306-order:pay_result_callback:</code></td>
      <td>Consuming/consumed marker<br><code>String</code></td>
      <td>Uses <code>SET NX GET PX</code> to claim processing and detect repeated MQ delivery.</td>
      <td>600 seconds while consuming; studied consumers set 7,200 seconds after success. Processing failure deletes the marker.</td>
      <td><code>IdempotentSpELByMQExecuteHandler</code>, annotated RocketMQ consumers</td>
    </tr>
    <tr>
      <td>Business services and shared frameworks</td>
      <td>Cache rebuild locks, registration/deletion locks, purchase locks, order-state locks, and REST idempotency locks</td>
      <td>Redisson-managed lock state</td>
      <td>Coordinates critical sections across JVMs. V1 purchases lock per train; V2 uses fair locks per train and seat type, preceded by local fair locks.</td>
      <td>Normally released after the critical section; Redisson manages the lock lease rather than the cache wrapper's value timeout.</td>
      <td><code>StringRedisTemplateProxy</code>, service implementations, <code>IdempotentSpELByRestAPIExecuteHandler</code></td>
    </tr>
  </tbody>
</table>

1. Purchase admission
   - [`TicketAvailabilityTokenBucket`][bucket] is inventory-based admission control. Its tokens represent available seats; there is no time-based refill rate.
   - The [Lua reservation script][bucket-lua] makes its Redis check and decrement one operation. MySQL seat updates and remote order creation occur afterward and are not part of that atomic operation.
   - V2 purchase exceptions do not have a general token-restoration block in `purchaseTicketsV2`; the explicit reversal paths cover cancellation and delayed closure.
2. Idempotency and locking
   - REST SpEL idempotency uses a lock while the annotated method runs; it does not retain a completed-response record.
   - [MQ idempotency][mq-idempotency] retains a processing/completion marker for a bounded interval.
   - The shared [token handler][token-idempotency] also supports `idempotent:token:<UUID>` with an empty String value, consumed by deletion. No business method selects the TOKEN strategy at this revision. Its fallback timeout is the raw value `6000`, interpreted in the host cache wrapper's configured unit.
   - Calls such as `lock()` and `tryLock()` omit an explicit lease and use Redisson's expiration-renewal path. This is library behavior, not custom renewal logic in 12306. [Redisson 3.21.3 lock implementation](https://github.com/redisson/redisson/blob/redisson-3.21.3/redisson/src/main/java/org/redisson/RedissonLock.java).

### 2.3 Cache Protection and Consistency

> Application logic that controls cache misses and keeps cached data aligned with database changes.

1. Cache penetration protection 缓存穿透
   - Problem: repeated requests for nonexistent data reach MySQL because there is no cached result.
   - Username checks: [`UserLoginServiceImpl.hasUsername`][user-login] uses the registration Bloom filter first. An absent name is considered available; a possibly present name is checked against the reusable-name Set. Registration still relies on database uniqueness constraints to reject duplicates.
   - A Bloom false positive without a reusable-name entry is treated as unavailable; this availability check has no database fallback to correct it.
   - Ticket-query validation: [`TrainTicketQueryParamVerifyChainFilter`][query-filter] loads the full code-to-name Hash and rejects unknown region/station codes from that snapshot.
   - Limits: the generic `safeGet` API accepts a Bloom filter and miss callbacks, but the passenger and train lookups shown above do not pass them. Its loader does not store null/empty-string results, so repeated nonexistent train IDs or empty passenger lists can still reach MySQL.
   - The registration filter is initialized with `tryInit` and updated during registration. No checked-in startup rebuild from all existing users was found.
2. Cache breakdown protection 缓存击穿
   - Problem: many callers miss the same hot key and rebuild it concurrently.
   - [`StringRedisTemplateProxy.safeGet`][cache-proxy] uses this sequence:
     1. Read Redis and return a hit.
     2. Acquire `safe_get_distributed_lock_get:<key>`.
     3. Read Redis again after acquiring the lock.
     4. If still missing, load the database and cache a nonempty result.
     5. Release the lock.
   - Hash loaders implement the same lock-and-recheck pattern for station mappings, train lists, carriage numbers, and remaining tickets. The token-bucket loader uses `tryLock` and rejects contenders while initialization is in progress.
   - A lock does not create a negative cache entry: if the database repeatedly returns nothing, waiting callers may still reload it one after another.
3. Cache avalanche protection 缓存雪崩 — limited
   - Problem: many keys become unavailable together, causing a broad increase in database work.
   - Preload jobs and miss locks reduce cold-cache pressure for some paths.
   - Expiry values are fixed; no randomized TTL spread was found. There is no Redis-outage fallback or global rebuild-concurrency guard in the shared cache proxy. V2's cache-dependent query path also assumes the required entries exist.
4. Database-cache consistency — limited
   - Passenger records: [`PassengerServiceImpl`][passenger] changes MySQL and then deletes the passenger-list key. The next read rebuilds it.
   - Remaining tickets, default mode: the [purchase template][purchase-template] decrements route Hash fields; cancellation and delayed closure restore counts and tokens in separate steps.
   - Remaining tickets, optional binlog mode: setting `ticket.availability.cache-update.type=binlog` disables those direct cache-update paths. The [Canal consumer][canal-consumer] dispatches database changes to the [seat-cache handler][canal-seat]; order-close changes trigger seat unlocking and token restoration.
   - The checked-in YAML does not select binlog mode. Redis operations, database transactions, and remote order calls are not one atomic transaction; the code does not establish strict consistency or a complete repair process for every partial failure.
5. Cache pollution control — not covered
   - No application limit on the number of cached search keys was found. The station-search cache can create keys from user-supplied name prefixes.
   - Redis server eviction settings are also absent. Having a TTL on some values does not establish a complete pollution-control strategy.

[cache-config]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/frameworks/cache/src/main/java/org/opengoofy/index12306/framework/starter/cache/config/CacheAutoConfiguration.java
[cache-proxy]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/frameworks/cache/src/main/java/org/opengoofy/index12306/framework/starter/cache/StringRedisTemplateProxy.java
[redis-properties]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/frameworks/cache/src/main/java/org/opengoofy/index12306/framework/starter/cache/config/RedisDistributedProperties.java
[bloom-config]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/user-service/src/main/java/org/opengoofy/index12306/biz/userservice/config/UserRegisterBloomFilterProperties.java
[user-login]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/user-service/src/main/java/org/opengoofy/index12306/biz/userservice/service/impl/UserLoginServiceImpl.java
[passenger]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/user-service/src/main/java/org/opengoofy/index12306/biz/userservice/service/impl/PassengerServiceImpl.java
[ticket]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/service/impl/TicketServiceImpl.java
[seat-margin]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/service/cache/SeatMarginCacheLoader.java
[query-filter]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/service/handler/ticket/filter/query/TrainTicketQueryParamVerifyChainFilter.java
[bucket]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/service/handler/ticket/tokenbucket/TicketAvailabilityTokenBucket.java
[bucket-lua]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/resources/lua/ticket_availability_token_bucket.lua
[mq-idempotency]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/frameworks/idempotent/src/main/java/org/opengoofy/index12306/framework/starter/idempotent/core/spel/IdempotentSpELByMQExecuteHandler.java
[token-idempotency]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/frameworks/idempotent/src/main/java/org/opengoofy/index12306/framework/starter/idempotent/core/token/IdempotentTokenExecuteHandler.java
[region-job]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/job/RegionTrainStationJobHandler.java
[remaining-job]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/job/TrainStationRemainingTicketJobHandler.java
[purchase-template]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/service/handler/ticket/base/AbstractTrainPurchaseTicketTemplate.java
[canal-consumer]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/mq/consumer/CanalCommonSyncBinlogConsumer.java
[canal-seat]: https://github.com/nageoffer/12306/blob/daf3a2714c00ff263764d23ab10c2eaed2ae842d/services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/canal/TicketAvailabilityCacheUpdateHandler.java
