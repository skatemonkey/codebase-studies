# Concurrency and Locking

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Concurrency](#1-concurrency)
> - [2. Locking](#2-locking)

## 0. Overview

1. Coverage
   - Concurrency: 12 use cases.
   - Locking: 3 use cases.

2. Mechanisms
   - Concurrency mechanisms
     - [Shared `ThreadPoolExecutor`](01-shared-thread-pool-executor.md)
       - [`CompletableFuture`](01-shared-thread-pool-executor.md#completable-future)
       - [`Runnable` through `execute(...)`](01-shared-thread-pool-executor.md#runnable)
     - Spring `@Async`: [Admin audit logging](../03-admin-audit-logging.md)
     - [`TaskScheduler`](02-scheduled-jobs.md)
       - `@Scheduled`
     - [Manually created `Thread`](03-manually-created-threads.md)
   - Locking mechanisms
     - [Redis `setIfAbsent` lock](04-locking.md#redis-setifabsent-lock)
     - [`synchronized (this)`](04-locking.md#synchronized-this)
     - [`synchronized` method](04-locking.md#synchronized-method)

## 1. Concurrency

<table>
  <thead>
    <tr><th>Group</th><th>Mechanism</th><th>Function</th><th>Use case</th></tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="3">Request-time parallel work</td>
      <td rowspan="3">Executor: <a href="01-shared-thread-pool-executor.md">Shared <code>ThreadPoolExecutor</code></a><br>Task: <a href="01-shared-thread-pool-executor.md#completable-future"><code>CompletableFuture</code></a></td>
      <td><code>PageController.index</code></td>
      <td>Renders the homepage by loading:<ul><li>Book settings</li><li>News</li></ul></td>
    </tr>
    <tr>
      <td><code>PageController.bookDetail</code></td>
      <td>Renders the book-detail page by loading:<ul><li>Book information</li><li>Comments</li><li>First chapter ID</li><li>Recommendations</li></ul></td>
    </tr>
    <tr>
      <td><code>PageController.bookContent</code></td>
      <td>Renders the chapter-reading page by loading:<ul><li>Book information</li><li>Chapter information</li><li>Previous chapter ID</li><li>Next chapter ID</li><li>Chapter content</li><li>Purchase requirement</li></ul></td>
    </tr>
    <tr>
      <td rowspan="2">Background asynchronous work</td>
      <td>Executor: <a href="01-shared-thread-pool-executor.md">Shared <code>ThreadPoolExecutor</code></a><br>Task: <a href="01-shared-thread-pool-executor.md#runnable"><code>Runnable</code> through <code>execute(...)</code></a></td>
      <td><a href="../flows/05-ai-cover-generation.md"><code>BookServiceImpl.addBook</code></a></td>
      <td>Generates a missing book cover.</td>
    </tr>
    <tr>
      <td>Spring <code>@Async</code></td>
      <td><a href="../03-admin-audit-logging.md"><code>LogServiceImpl.save</code></a></td>
      <td>Saves an admin audit log.</td>
    </tr>
    <tr>
      <td rowspan="3">Continuous threads</td>
      <td rowspan="3"><a href="03-manually-created-threads.md">Manually created <code>Thread</code></a></td>
      <td rowspan="2"><code>StarterListener.onStartup</code></td>
      <td>Updates eligible books.</td>
    </tr>
    <tr>
      <td>Processes single-book crawl tasks.</td>
    </tr>
    <tr>
      <td><code>CrawlServiceImpl.openOrCloseCrawl</code></td>
      <td>Crawls enabled categories.</td>
    </tr>
    <tr>
      <td rowspan="4">Scheduled jobs</td>
      <td rowspan="4">Scheduler: <a href="02-scheduled-jobs.md"><code>TaskScheduler</code></a><br>Task: <code>@Scheduled</code></td>
      <td><code>DailyIncomeStaSchedule.statistics</code></td>
      <td>Calculates daily author income.</td>
    </tr>
    <tr>
      <td><code>MonthIncomeStaSchedule.statistics</code></td>
      <td>Calculates monthly author income.</td>
    </tr>
    <tr>
      <td><code>CrawlPicTransSchedule.trans</code></td>
      <td>Transfers crawler images.</td>
    </tr>
    <tr>
      <td><code>DiskMonitorSchedule.checkDiskUsage</code></td>
      <td>Checks disk usage.</td>
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
      <td>Distributed locking</td>
      <td><a href="04-locking.md#redis-setifabsent-lock">Redis <code>setIfAbsent</code> lock</a></td>
      <td><a href="../flows/01-book-download.md"><code>BookController.download</code></a></td>
      <td>Prevents duplicate book downloads.</td>
    </tr>
    <tr>
      <td rowspan="2">JVM locking</td>
      <td><a href="04-locking.md#synchronized-this"><code>synchronized (this)</code></a></td>
      <td><code>StarterListener.onStartup</code></td>
      <td>Serializes book batch selection.</td>
    </tr>
    <tr>
      <td><a href="04-locking.md#synchronized-method"><code>synchronized</code> method</a></td>
      <td><code>IdWorker.nextId</code></td>
      <td>Serializes ID generation.</td>
    </tr>
  </tbody>
</table>
