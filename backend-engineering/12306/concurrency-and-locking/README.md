# Concurrency and Locking

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [12306 overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Concurrency](#1-concurrency)
>   - [1.1 Task Execution](#11-task-execution)
>   - [1.2 Thread State](#12-thread-state)
> - [2. Locking](#2-locking)
>   - [2.1 JVM Locking](#21-jvm-locking)
>   - [2.2 Distributed Locking](#22-distributed-locking)
>   - [2.3 Atomic Redis Operations](#23-atomic-redis-operations)
>   - [2.4 Implementation Caveats](#24-implementation-caveats)

## 0. Overview

1. Coverage
   - Concurrency: 15 use cases.
   - Locking: 22 use cases.

2. Mechanisms
   - Concurrency
     - [Hippo4j `ThreadPoolExecutor`](01-hippo4j-thread-pool-executor.md)
       - `Callable` through `submit(...)`
     - [`ScheduledThreadPoolExecutor`](#scheduled-thread-pool-executor)
       - `Runnable` through `schedule(...)`
     - [RocketMQ consumer threads](#rocketmq-consumer-threads)
     - [XXL-JOB](#xxl-job)
     - [Thread State](#12-thread-state)
   - Locking
     - JVM: `synchronized`, fair `ReentrantLock`.
     - Redis: Redisson locks and Lua scripts.

## 1. Concurrency

### 1.1 Task Execution

<table>
  <thead>
    <tr><th>Group</th><th>Mechanism</th><th>Function</th><th>Use case</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>Request parallelism</td>
      <td>Executor: <a href="01-hippo4j-thread-pool-executor.md">Hippo4j <code>ThreadPoolExecutor</code></a><br>Task: <code>submit(Callable)</code><br>Result: <code>Future.get()</code></td>
      <td><code>TrainSeatTypeSelector.select</code></td>
      <td>Allocate seats in parallel by seat type.</td>
    </tr>
    <tr>
      <td>Delayed work</td>
      <td>Executor: <a href="#scheduled-thread-pool-executor"><code>ScheduledThreadPoolExecutor</code></a><br>Task: <code>schedule(Runnable)</code></td>
      <td><code>TicketServiceImpl.tokenIsNullRefreshToken</code></td>
      <td>Recheck inventory after token shortages.</td>
    </tr>
    <tr>
      <td rowspan="5">MQ consumers</td>
      <td rowspan="5">Executor: <a href="#rocketmq-consumer-threads">RocketMQ consumer threads</a><br>Task: <code>RocketMQListener.onMessage</code></td>
      <td><code>DelayCloseOrderConsumer.onMessage</code></td>
      <td>Close unpaid orders; release stock.</td>
    </tr>
    <tr>
      <td><code>PayResultCallbackTicketConsumer.onMessage</code></td>
      <td>Mark seats as sold.</td>
    </tr>
    <tr>
      <td><code>PayResultCallbackOrderConsumer.onMessage</code></td>
      <td>Update order payment state.</td>
    </tr>
    <tr>
      <td><code>RefundResultCallbackOrderConsumer.onMessage</code></td>
      <td>Update order refund state.</td>
    </tr>
    <tr>
      <td><code>CanalCommonSyncBinlogConsumer.onMessage</code></td>
      <td>Refresh caches from binlog (optional).</td>
    </tr>
    <tr>
      <td rowspan="4">Deprecated jobs</td>
      <td rowspan="4">Scheduler: <a href="#xxl-job">XXL-JOB</a><br>Task: <code>@XxlJob</code></td>
      <td><code>TrainStationJobHandler.execute</code></td>
      <td>Preload train stops.</td>
    </tr>
    <tr>
      <td><code>TrainStationRemainingTicketJobHandler.execute</code></td>
      <td>Preload demo inventory.</td>
    </tr>
    <tr>
      <td><code>TrainStationDetailJobHandler.execute</code></td>
      <td>Preload legacy train details.</td>
    </tr>
    <tr>
      <td><code>RegionTrainStationJobHandler.execute</code></td>
      <td>Preload train lists by date.</td>
    </tr>
  </tbody>
</table>

1. <a name="scheduled-thread-pool-executor"></a>`ScheduledThreadPoolExecutor`
   - Use: delayed inventory rechecks.
   - Instance: `TicketServiceImpl.tokenIsNullRefreshExecutor`.
   - Creation: `Executors.newScheduledThreadPool(1)`.
   - Task: `TicketServiceImpl.tokenIsNullRefreshToken`; runs after 10 seconds.
   - Lock ownership: [implementation caveat](#24-implementation-caveats).
2. <a name="rocketmq-consumer-threads"></a>RocketMQ consumer threads
   - RocketMQ manages these threads; no custom count is configured.
   - Binlog mode requires `ticket.availability.cache-update.type=binlog`; off in checked-in YAML.
   - Default order closure restores inventory directly.
3. <a name="xxl-job"></a>XXL-JOB
   - Use: deprecated cache-preload jobs.
   - All four XXL-JOB handlers expose HTTP endpoints. No enabled schedule was verified.
   - The region preload job writes Sorted Sets; current queries read separate Hashes.
4. Unused utilities
   - Local `ThreadPoolBuilder` and `EagerThreadPoolExecutor`: no business caller found. Seat selection uses Hippo4j.
   - `SurplusNeedMatchSeatUtil.getSurplusNeedMatchSeat`: unused `parallelStream()` helper.

### 1.2 Thread State

<table>
  <thead>
    <tr><th>Mechanism</th><th>Function</th><th>Purpose</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><code>Collections.synchronizedList</code></td>
      <td><code>TrainSeatTypeSelector.select</code></td>
      <td>Protect parallel result collection.</td>
    </tr>
    <tr>
      <td><code>AtomicBoolean.compareAndSet</code></td>
      <td><code>ApplicationContentPostProcessor.onApplicationEvent</code></td>
      <td>Publish initialization once per instance.</td>
    </tr>
    <tr>
      <td><code>TransmittableThreadLocal</code></td>
      <td><code>UserContext</code><br><code>UserTransmitFilter.doFilter</code></td>
      <td>Store the current request user.</td>
    </tr>
    <tr>
      <td><code>ThreadLocal</code></td>
      <td><code>IdempotentContext</code><br><code>IdempotentAspect.idempotentHandler</code></td>
      <td>Store the current invocation state.</td>
    </tr>
  </tbody>
</table>

- Both contexts are cleared in `finally`. Cross-thread propagation needs executor support.
- Seat-handler `AtomicInteger` values are local stream counters.

## 2. Locking

### 2.1 JVM Locking

<table>
  <thead>
    <tr><th>Mechanism</th><th>Function</th><th>Use case</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><code>synchronized (TicketService.class)</code></td>
      <td rowspan="2"><code>TicketServiceImpl.purchaseTicketsV2</code></td>
      <td>Initialize purchase locks and refresh markers.</td>
    </tr>
    <tr>
      <td><code>new ReentrantLock(true)</code></td>
      <td>Serialize purchases per train/seat type.</td>
    </tr>
    <tr>
      <td rowspan="2"><code>synchronized</code> method</td>
      <td><code>Snowflake.nextId</code></td>
      <td>Protect Snowflake timestamp and sequence.</td>
    </tr>
    <tr>
      <td><code>DistributedIdGenerator.generateId</code> in order-service<br><code>DistributedIdGenerator.generateId</code> in pay-service</td>
      <td>Protect order/payment ID generation.</td>
    </tr>
  </tbody>
</table>

### 2.2 Distributed Locking

<table>
  <thead>
    <tr><th>Group</th><th>Mechanism</th><th>Function</th><th>Use case</th></tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="2">Purchases</td>
      <td>Redisson <code>getLock(...).lock()</code></td>
      <td><code>TicketServiceImpl.purchaseTicketsV1</code></td>
      <td>Serialize purchases per train.</td>
    </tr>
    <tr>
      <td>Redisson <code>getFairLock(...).lock()</code></td>
      <td><code>TicketServiceImpl.purchaseTicketsV2</code></td>
      <td>Serialize purchases per train/seat type.</td>
    </tr>
    <tr>
      <td rowspan="2">Accounts</td>
      <td>Redisson <code>tryLock()</code></td>
      <td><code>UserLoginServiceImpl.register</code></td>
      <td>Prevent concurrent registration per username.</td>
    </tr>
    <tr>
      <td>Redisson <code>lock()</code></td>
      <td><code>UserLoginServiceImpl.deletion</code></td>
      <td>Serialize account deletion per username.</td>
    </tr>
    <tr>
      <td>REST idempotency</td>
      <td>Redisson <code>tryLock()</code></td>
      <td><code>IdempotentSpELByRestAPIExecuteHandler.handler</code></td>
      <td>Reject overlapping requests with the same key.</td>
    </tr>
    <tr>
      <td rowspan="6">Cache rebuilding</td>
      <td rowspan="6">Redisson <code>lock()</code> + recheck</td>
      <td><code>StringRedisTemplateProxy.safeGet</code></td>
      <td>Rebuild missing cache entries per key.</td>
    </tr>
    <tr>
      <td><code>TicketServiceImpl.pageListTicketQueryV1</code></td>
      <td>Rebuild station mappings and train lists.</td>
    </tr>
    <tr>
      <td><code>TrainTicketQueryParamVerifyChainFilter.handler</code></td>
      <td>Load valid station/region codes.</td>
    </tr>
    <tr>
      <td><code>RegionStationImpl.safeGetRegionStation</code></td>
      <td>Load station-search results per key.</td>
    </tr>
    <tr>
      <td><code>CarriageServiceImpl.safeGetCarriageNumber</code></td>
      <td>Load carriage numbers per train.</td>
    </tr>
    <tr>
      <td><code>SeatMarginCacheLoader.load</code></td>
      <td>Load remaining-seat counts.</td>
    </tr>
    <tr>
      <td>Inventory initialization</td>
      <td>Redisson <code>tryLock()</code> + recheck</td>
      <td><code>TicketAvailabilityTokenBucket.takeTokenFromBucket</code></td>
      <td>Initialize a train’s token bucket.</td>
    </tr>
    <tr>
      <td>Inventory recheck</td>
      <td>Redisson <code>tryLock()</code></td>
      <td><code>TicketServiceImpl.tokenIsNullRefreshToken</code></td>
      <td>Recheck token shortages per train.</td>
    </tr>
    <tr>
      <td rowspan="2">Order state</td>
      <td rowspan="2">Redisson <code>tryLock()</code></td>
      <td><code>OrderServiceImpl.cancelTickOrder</code></td>
      <td>Prevent concurrent order cancellation.</td>
    </tr>
    <tr>
      <td><code>OrderServiceImpl.statusReversal</code><br><code>OrderItemServiceImpl.orderItemStatusReversal</code></td>
      <td>Update payment/refund state.</td>
    </tr>
    <tr>
      <td>Generator initialization</td>
      <td>Redisson <code>lock()</code></td>
      <td><code>OrderIdGeneratorManager.afterPropertiesSet</code><br><code>PayIdGeneratorManager.afterPropertiesSet</code></td>
      <td>Allocate generator nodes at startup.</td>
    </tr>
  </tbody>
</table>

- REST locks cover purchase, passenger changes, and payment; released after execution.
- Token-bucket initialization rejects lock contenders. V2 takes local locks before Redis locks.
- Delayed order closure calls `cancelTickOrder` through `closeTickOrder`.
- Cache keys: [Redis Integration](../01-redis-integration.md#2-usage).

### 2.3 Atomic Redis Operations

<table>
  <thead>
    <tr><th>Mechanism</th><th>Function</th><th>Use case</th></tr>
  </thead>
  <tbody>
    <tr>
      <td>Lua check and decrement</td>
      <td><code>TicketAvailabilityTokenBucket.takeTokenFromBucket</code></td>
      <td>Check and reserve inventory tokens.</td>
    </tr>
    <tr>
      <td>Lua <code>SET NX GET PX</code></td>
      <td><code>IdempotentSpELByMQExecuteHandler</code></td>
      <td>Claim messages and suppress duplicate processing.</td>
    </tr>
  </tbody>
</table>

- Token returns use a separate rollback script.
- MQ marker: 600 seconds during processing; 7,200 after success; deleted on failure.
- Redis atomicity excludes MySQL writes and remote calls.

### 2.4 Implementation Caveats

1. Order state: both status-reversal methods continue after failed `tryLock()`, then attempt `unlock()`.
2. Delayed task: `tokenIsNullRefreshToken` acquires on the calling thread and unlocks on the scheduled thread. Release is unverified.
