# Concurrency and Locking

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [Jeepay overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Concurrency](#1-concurrency)
> - [2. Locking](#2-locking)

## 0. Overview

1. Coverage
   - Concurrency: 15 use cases.
   - Locking: 17 use cases.

2. Mechanisms
   - Concurrency mechanisms
     - `ScheduledThreadPoolExecutor`
       - `jeepay-manager` and `jeepay-merchant` each create their own pool with core size **10**. Up to 10 log-saving tasks run concurrently per pool; additional tasks wait in the queue.
       - Created by `Executors.newScheduledThreadPool(10)` per application.
       - Extends `ThreadPoolExecutor`.
       - `Runnable` through `execute(...)`
     - Spring `@Async`
     - [Custom Spring `ThreadPoolTaskExecutor` + `@Async`](02-custom-thread-pool-task-executor.md)
       - Configured by [`MqThreadExecutor`](02-custom-thread-pool-task-executor.md#creation-and-configuration).
     - Default `RocketMQ` thread pool
       - Created automatically per consumer.
     - [`TaskScheduler`](01-scheduled-jobs.md)
       - `@Scheduled`
   - Locking mechanisms
     - JVM locking
       - `synchronized` method
       - `static synchronized` method
     - [Database concurrency control](03-database-concurrency-control.md)
       - Expected-state SQL predicates
       - SQL increments with amount predicates
       - Spring `@Transactional`
       - Database unique keys

## 1. Concurrency

<table>
  <thead>
    <tr><th>Group</th><th>Mechanism</th><th>Function</th><th>Use case</th></tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">Background asynchronous work</td>
      <td rowspan="2">Executor: <code>ScheduledThreadPoolExecutor</code><br>Created by: <code>Executors.newScheduledThreadPool(10)</code> per application<br>Extends: <code>ThreadPoolExecutor</code><br>Task: <code>Runnable</code> through <code>execute(...)</code></td>
      <td rowspan="2"><a href="../01-operation-audit-logging.md"><code>MethodLogAop</code></a><br><code>jeepay-manager</code><br><code>jeepay-merchant</code></td>
      <td>Saves platform operation logs.</td>
    </tr>
    <tr>
      <td>Saves merchant operation logs.</td>
    </tr>
    <tr>
      <td>Spring <code>@Async</code></td>
      <td><code>SysConfigController.updateSysConfigMQ</code></td>
      <td>Publishes configuration-refresh messages.</td>
    </tr>
    <tr>
      <td rowspan="6"><a href="../02-message-queue.md">MQ message handling</a></td>
      <td rowspan="2"><a href="02-custom-thread-pool-task-executor.md">Custom Spring <code>ThreadPoolTaskExecutor</code> + <code>@Async</code></a></td>
      <td><a href="02-custom-thread-pool-task-executor.md#tasks"><code>PayOrderMchNotifyMQReceiver.receive</code></a></td>
      <td>Delivers merchant callbacks and retries failures.</td>
    </tr>
    <tr>
      <td><a href="02-custom-thread-pool-task-executor.md#tasks"><code>PayOrderReissueMQReceiver.receive</code></a></td>
      <td>Polls pending payment results.</td>
    </tr>
    <tr>
      <td rowspan="4">Default <code>RocketMQ</code> thread pool</td>
      <td><code>PayOrderDivisionMQReceiver.receive</code></td>
      <td>Processes queued profit-sharing tasks.</td>
    </tr>
    <tr>
      <td><code>ResetAppConfigMQReceiver.receive</code><br>all three applications</td>
      <td>Reloads system configuration from broadcast messages.</td>
    </tr>
    <tr>
      <td><code>ResetIsvMchAppInfoMQReceiver.receive</code></td>
      <td>Reloads merchant and channel configuration from broadcast messages.</td>
    </tr>
    <tr>
      <td><code>CleanMchLoginAuthCacheMQReceiver.receive</code></td>
      <td>Clears merchant login caches.</td>
    </tr>
    <tr>
      <td rowspan="6">Scheduled jobs</td>
      <td rowspan="6">Scheduler: <a href="01-scheduled-jobs.md"><code>TaskScheduler</code></a><br>Task: <code>@Scheduled</code></td>
      <td><code>PayOrderReissueTask.start</code></td>
      <td>Queries payments pending for at least ten minutes.</td>
    </tr>
    <tr>
      <td><code>RefundOrderReissueTask.start</code></td>
      <td>Queries pending refund results.</td>
    </tr>
    <tr>
      <td><code>TransferOrderReissueTask.start</code></td>
      <td>Queries pending transfers created within the past day.</td>
    </tr>
    <tr>
      <td><code>PayOrderDivisionRecordReissueTask.start</code></td>
      <td>Queries profit-sharing records pending for five minutes.</td>
    </tr>
    <tr>
      <td><code>PayOrderExpiredTask.start</code></td>
      <td>Closes expired payment orders.</td>
    </tr>
    <tr>
      <td><code>RefundOrderExpiredTask.start</code></td>
      <td>Closes expired refund orders.</td>
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
      <td rowspan="9">JVM locking</td>
      <td><code>synchronized</code> method</td>
      <td><code>ConfigContextService.initMchInfoConfigContext</code></td>
      <td>Serializes merchant cache reloads.</td>
    </tr>
    <tr>
      <td><code>synchronized</code> method</td>
      <td><code>ConfigContextService.initMchAppConfigContext</code></td>
      <td>Serializes application cache reloads.</td>
    </tr>
    <tr>
      <td><code>synchronized</code> method</td>
      <td><code>ConfigContextService.initIsvConfigContext</code></td>
      <td>Serializes service provider cache reloads.</td>
    </tr>
    <tr>
      <td><code>synchronized</code> method</td>
      <td><code>SysConfigService.initDBConfig</code></td>
      <td>Serializes system configuration reloads.</td>
    </tr>
    <tr>
      <td><code>synchronized</code> method</td>
      <td><code>ChannelCertConfigKitBean.downloadFile</code></td>
      <td>Serializes certificate downloads.</td>
    </tr>
    <tr>
      <td><code>synchronized</code> method</td>
      <td><code>ActiveMQConfig.init</code></td>
      <td>Serializes ActiveMQ destination creation.</td>
    </tr>
    <tr>
      <td><code>static synchronized</code> method</td>
      <td><code>WsPayOrderServer</code></td>
      <td>Serializes payment client counter access.</td>
    </tr>
    <tr>
      <td><code>static synchronized</code> method</td>
      <td><code>WsTransferOrderServer</code></td>
      <td>Serializes transfer client counter access.</td>
    </tr>
    <tr>
      <td><code>static synchronized</code> method</td>
      <td><code>WsChannelUserIdServer</code></td>
      <td>Serializes authorization client counter access.</td>
    </tr>
    <tr>
      <td rowspan="8">Database concurrency control</td>
      <td>Expected-state SQL predicates</td>
      <td><code>PayOrderService</code></td>
      <td>Guards payment-state transitions.</td>
    </tr>
    <tr>
      <td>Expected-state SQL predicates<br>Spring <code>@Transactional</code></td>
      <td><code>RefundOrderService</code></td>
      <td>Guards refund-state transitions and payment-total updates.</td>
    </tr>
    <tr>
      <td>Expected-state SQL predicates<br>Spring <code>@Transactional</code></td>
      <td><code>TransferOrderService</code></td>
      <td>Guards transfer-state transitions.</td>
    </tr>
    <tr>
      <td>SQL increments with amount predicates</td>
      <td><code>PayOrderMapper.updateRefundAmountAndCount</code></td>
      <td>Updates refund totals without exceeding the payment amount.</td>
    </tr>
    <tr>
      <td>Expected-state SQL predicates</td>
      <td><code>PayOrderDivisionProcessService.processPayOrderDivision</code></td>
      <td>Lets one worker claim a profit-sharing order.</td>
    </tr>
    <tr>
      <td>Expected-state SQL predicates<br>Spring <code>@Transactional</code></td>
      <td><code>PayOrderDivisionRecordService.updateResendState</code></td>
      <td>Resets failed profit-sharing records for retry.</td>
    </tr>
    <tr>
      <td>Expected-state SQL predicates</td>
      <td><code>PayOrderDivisionRecordService.updateRecordSuccessOrFail</code><br><code>updateRecordSuccessOrFailBySingleItem</code></td>
      <td>Guards profit-sharing result updates.</td>
    </tr>
    <tr>
      <td>Database unique keys</td>
      <td>Order and notification inserts</td>
      <td>Rejects duplicate merchant order numbers and notification records.</td>
    </tr>
  </tbody>
</table>
