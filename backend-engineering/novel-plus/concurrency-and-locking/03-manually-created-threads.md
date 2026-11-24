# Manually Created Threads

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [1. Thread Groups](#thread-groups)
> - [2. Automatic Book-Update Threads](#automatic-book-update-threads)
> - [3. Single-Book Task Thread](#single-book-task-thread)
> - [4. Category Crawler Threads](#category-crawler-threads)

1. <a name="thread-groups"></a>Thread groups

   <table>
     <thead>
       <tr><th>Created by</th><th>Thread use case</th><th>When created</th><th>Threads created</th><th>Repeated work</th><th>Stops when</th></tr>
     </thead>
     <tbody>
       <tr>
         <td rowspan="2"><code>StarterListener.onStartup</code></td>
         <td><a href="#automatic-book-update-threads">Automatic book-update threads</a></td>
         <td><code>novel-crawl</code> starts</td>
         <td>1 with the current <code>crawl.update.thread=1</code> configuration</td>
         <td>Update existing books → sleep 10 minutes</td>
         <td>Application ends</td>
       </tr>
       <tr>
         <td><a href="#single-book-task-thread">Single-book task thread</a></td>
         <td><code>novel-crawl</code> starts</td>
         <td>1</td>
         <td>Check queue → process one book → sleep 1 minute</td>
         <td>Application ends</td>
       </tr>
       <tr>
         <td><code>CrawlServiceImpl.openOrCloseCrawl</code></td>
         <td><a href="#category-crawler-threads">Category crawler threads</a></td>
         <td>A crawler source is enabled</td>
         <td>7 per source</td>
         <td>Each thread crawls one category repeatedly</td>
         <td>Source is disabled</td>
       </tr>
     </tbody>
   </table>

2. <a name="automatic-book-update-threads"></a>Automatic book-update threads
   1. Creation
      - When `novel-crawl` starts, `StarterListener.onStartup` uses a `for` loop to create the number of Java threads set by `crawl.update.thread`; its current value is `1`.
   2. Continuous loop
      1. Select: Inside `synchronized (this)`, select up to 100 books and update their `crawl_last_time`.
      2. Update: Crawl and save every selected book.
      3. Wait: Sleep for 10 minutes, then repeat.
3. <a name="single-book-task-thread"></a>Single-book task thread
   1. Creation
      - When `novel-crawl` starts, `StarterListener.onStartup` directly creates one Java thread.
   2. Continuous loop
      1. Select: Get one task—the oldest waiting task—from `crawl_single_task`.
      2. Process: Crawl and save the requested book.
      3. Wait: Sleep for one minute, record the result, then repeat.
4. <a name="category-crawler-threads"></a>Category crawler threads
   1. Creation
      - When a crawler source is enabled, `CrawlServiceImpl.openOrCloseCrawl` uses a `for` loop to create seven Java threads—one for each category.
   2. Continuous loop
      1. Crawl: Crawl and save the books on the assigned category page.
      2. Advance: Continue through the category's remaining pages.
      3. Repeat: After the final page, return to page one, sleep for one minute, and repeat until the source is disabled.
