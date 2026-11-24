# Shared Thread Pool Executor

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [1. Thread Pool Setup](#thread-pool-setup)
> - [2. Thread Pool Usage](#thread-pool-usage)
> - [3. Task Submission Methods](#task-submission-methods)
>   - [3.1 `CompletableFuture`](#completable-future)
>   - [3.2 `Runnable`](#runnable)

1. <a name="thread-pool-setup"></a>Thread Pool Setup
   1. Creation
      - `ThreadPoolConfig.threadPoolExecutor()` creates one `ThreadPoolExecutor` bean in the `novel-front` application context; its consumers submit tasks to that bean.
      - Source file: `novel-front/src/main/java/com/java2nb/novel/core/config/ThreadPoolConfig.java`
   2. Configuration
      - Source files:
        - `novel-front/src/main/java/com/java2nb/novel/core/config/ThreadPoolProperties.java`
        - `novel-front/src/main/resources/application-dev.yml`
        - `novel-front/src/main/resources/application-prod.yml`
      - Core threads: 10
      - Maximum threads: 20
      - Keep-alive time: 10 seconds
      - Queue: `LinkedBlockingDeque`
        - Development capacity: 100
        - Production capacity: 1,000
      - Rejection policy: `AbortPolicy`
2. <a name="thread-pool-usage"></a>Thread Pool Usage

   > Two classes submit tasks to the shared pool:

   - `PageController`: Runs request-time parallel page-loading tasks.
   - `BookServiceImpl`: Runs missing-cover generation as a background task.
3. <a name="task-submission-methods"></a>Task Submission Methods
   1. <a name="completable-future"></a>`CompletableFuture`
      1. `supplyAsync`
         - Starts the book query and stores its returned `Book` in `bookCompletableFuture`.

         ```java
         CompletableFuture<Book> bookCompletableFuture = CompletableFuture.supplyAsync(() -> {
             Book book = bookService.queryBookDetail(bookId);
             log.debug("加载小说基本信息线程结束");
             return book;
         }, threadPoolExecutor);
         ```

      2. `thenApplyAsync`
         - Waits for `bookCompletableFuture`, then passes its `Book` result into the `book` lambda parameter.

         ```java
         CompletableFuture<Long> firstBookIndexIdCompletableFuture = bookCompletableFuture.thenApplyAsync((book) -> {
             if (book.getLastIndexId() != null) {
                 Long firstBookIndexId = bookService.queryFirstBookIndexId(bookId);
                 log.debug("加载小说基本信息线程结束");
                 return firstBookIndexId;
             }
             return null;
         }, threadPoolExecutor);
         ```

      3. Result
         - Returns a `CompletableFuture<T>`; `get()` returns `T` after the task finishes.
         - The request thread waits at `get()` before returning the template.
   2. <a name="runnable"></a>`Runnable`

      ```java
      threadPoolExecutor.execute(() -> { // Runnable task
          // 1. Build the AI prompt.
          // 2. Generate the cover.
          // 3. Download the generated image to local storage.
          // 4. Update the book's picUrl in MySQL.
          // 5. Cache the picUrl in Redis for one hour.
      });
      ```

      - Result: Returns no task result.
      - Caller: Continues without waiting for the task to finish.
