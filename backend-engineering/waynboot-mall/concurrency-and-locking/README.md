# Concurrency and Locking

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [waynboot-mall overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Concurrency](#1-concurrency)
> - [2. Locking](#2-locking)

## 0. Overview

> Task execution and explicit locks in [waynboot-mall](../README.md), verified at revision `c683afd`.

1. Coverage
   - [Concurrency](#1-concurrency): Request-time parallel work, background processing, and scheduled jobs.
   - [Locking](#2-locking): Operations guarded by Redis locks or JVM synchronization.

2. Mechanisms
   - Concurrency mechanisms
     - Shared `ThreadPoolTaskExecutor`: `commonThreadPoolTaskExecutor`.
       - `CompletableFuture` for homepage queries.
       - `Callable`, `Future`, and `FutureTask` for catalog queries.
       - `Runnable` through `execute(...)` for background work.
     - RabbitMQ `@RabbitListener` for message-driven work.
     - Spring `@Scheduled` for recurring application jobs.
     - `ScheduledExecutorService` for the delay-queue worker and Redis lock renewal.
   - Locking mechanisms
     - Custom Redis Lua lock: `RedisLock`, wrapped by `TradeLockSupport` for business operations.
     - Redis-backed ShedLock: `@SchedulerLock` for governance jobs.
     - JVM `synchronized` method for ID generation.

## 1. Concurrency

<table>
  <thead>
    <tr><th>Group</th><th>Mechanism</th><th>Function</th><th>Use case</th></tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">Request-time parallel work</td>
      <td>Shared <code>ThreadPoolTaskExecutor</code><br><code>CompletableFuture</code></td>
      <td><code>IHomeServiceImpl.index</code></td>
      <td>Loads the homepage's independent content blocks in parallel.</td>
    </tr>
    <tr>
      <td>Shared <code>ThreadPoolTaskExecutor</code><br><code>FutureTask</code></td>
      <td><code>GoodsDetailServiceImpl.getGoodsDetailData</code></td>
      <td>Loads specifications, SKU data, and attributes in parallel on a cache miss.</td>
    </tr>
    <tr>
      <td>Shared <code>ThreadPoolTaskExecutor</code><br><code>Callable</code> / <code>Future</code></td>
      <td><code>CategoryServiceImpl.index</code><br><code>CategoryServiceImpl.content</code><br>Shared helper: <code>loadCategoryContent</code></td>
      <td>Loads the current category and its children in parallel.</td>
    </tr>
    <tr>
      <td rowspan="4">Background asynchronous work</td>
      <td rowspan="3">Shared <code>ThreadPoolTaskExecutor</code><br><code>Runnable</code> through <code>execute(...)</code></td>
      <td><code>LoginService.refreshLastLoginAsync</code></td>
      <td>Updates the member's last-login information.</td>
    </tr>
    <tr>
      <td><code>SearchApplicationService.saveSearchHistoryAsync</code></td>
      <td>Saves search history after a successful product search.</td>
    </tr>
    <tr>
      <td><code>MailUtil.sendMail</code></td>
      <td>Sends email outside the calling request's main work.</td>
    </tr>
    <tr>
      <td>RabbitMQ <code>@RabbitListener</code></td>
      <td><code>OrderPayConsumer.process</code><br><code>OrderDelayConsumer.process</code><br><code>SeckillOrderConsumer.process</code><br><code>SeckillOrderDelayConsumer.process</code><br><code>EmailSendConsumer.process</code></td>
      <td>Delegates ordinary and flash-sale order creation, timeout cancellation, and email sending through callbacks to the mobile API.</td>
    </tr>
    <tr>
      <td>Continuous worker</td>
      <td>Single-thread <code>ScheduledThreadPoolExecutor</code><br><code>DelayQueue</code></td>
      <td><code>TaskService.init</code></td>
      <td>Starts a worker that waits for delayed tasks; no application calls to <code>addTask</code> were found.</td>
    </tr>
    <tr>
      <td rowspan="5">Scheduled jobs</td>
      <td rowspan="4">Spring <code>@Scheduled</code></td>
      <td><code>TradeGovernanceScheduledTask.refreshStockSnapshot</code></td>
      <td>Refreshes Redis stock snapshots.</td>
    </tr>
    <tr>
      <td><code>TradeGovernanceScheduledTask.reconcileInventory</code></td>
      <td>Checks frozen stock against inventory flows, with optional repair.</td>
    </tr>
    <tr>
      <td><code>TradeGovernanceScheduledTask.reconcilePaymentDaily</code></td>
      <td>Runs daily payment reconciliation.</td>
    </tr>
    <tr>
      <td><code>LocalMessageRelayTask.relayPendingMessages</code></td>
      <td>Processes pending local messages every five seconds.</td>
    </tr>
    <tr>
      <td>Single-thread <code>ScheduledExecutorService</code><br><code>scheduleWithFixedDelay(...)</code></td>
      <td><code>RedisLock.lock</code></td>
      <td>Starts a renewal task after acquiring a Redis lock.</td>
    </tr>
  </tbody>
</table>

## 2. Locking

<table>
  <thead>
    <tr><th>Group</th><th>Mechanism</th><th>Function</th><th>Use case</th></tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="9">Distributed locking</td>
      <td rowspan="5">Custom Redis Lua lock through <code>TradeLockSupport</code></td>
      <td><code>CartWriteSupport.add</code></td>
      <td>Serializes additions to the same user's cart.</td>
    </tr>
    <tr>
      <td><code>CartWriteSupport.changeNum</code></td>
      <td>Serializes cart quantity changes by user.</td>
    </tr>
    <tr>
      <td><code>CartWriteSupport.updateChecked</code></td>
      <td>Serializes cart selection changes by user.</td>
    </tr>
    <tr>
      <td><code>CouponAcquireSupport.receive</code></td>
      <td>Serializes claims for the same user and coupon template.</td>
    </tr>
    <tr>
      <td><code>OrderCancellationSupport.cancel</code></td>
      <td>Serializes cancellation attempts for the same order.</td>
    </tr>
    <tr>
      <td>Custom Redis Lua lock through <code>RedisLock</code></td>
      <td><code>GoodsElasticSyncSupport.syncGoodsToEs</code></td>
      <td>Serializes full rebuilds of the goods search index.</td>
    </tr>
    <tr>
      <td rowspan="3">Redis-backed ShedLock<br><code>@SchedulerLock</code></td>
      <td><code>TradeGovernanceScheduledTask.refreshStockSnapshot</code></td>
      <td>Guards stock-snapshot refresh across admin instances.</td>
    </tr>
    <tr>
      <td><code>TradeGovernanceScheduledTask.reconcileInventory</code></td>
      <td>Guards inventory reconciliation across admin instances.</td>
    </tr>
    <tr>
      <td><code>TradeGovernanceScheduledTask.reconcilePaymentDaily</code></td>
      <td>Guards payment reconciliation across admin instances.</td>
    </tr>
    <tr>
      <td>JVM locking</td>
      <td><code>synchronized</code> method</td>
      <td><code>IdUtil.SnowFlake.nextId</code></td>
      <td>Serializes ID-sequence updates on one generator instance.</td>
    </tr>
  </tbody>
</table>
