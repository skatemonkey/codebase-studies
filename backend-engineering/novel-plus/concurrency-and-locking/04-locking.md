# Locking

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [1. Redis `setIfAbsent` Lock](#redis-setifabsent-lock)
> - [2. `synchronized (this)`](#synchronized-this)
> - [3. `synchronized` Method](#synchronized-method)

1. <a name="redis-setifabsent-lock"></a>Redis `setIfAbsent` lock
   1. Function
      - [`BookController.download`](../flows/01-book-download.md)
   2. Lock
      - `setIfAbsent(...)` tries to create `bookIsDownloading:<bookId>`.
      - Returns `true`: Continue. Returns `false`: Reject the request.
   3. Boundary
      - Application instances sharing Redis also share this lock.
2. <a name="synchronized-this"></a>`synchronized (this)`
   1. Function
      - [`StarterListener.onStartup`](03-manually-created-threads.md#automatic-book-update-threads)
   2. Lock
      - Lets only one update thread call `queryNeedUpdateBook(...)` at a time.
   3. Boundary
      - Works only among threads using the same `StarterListener` object inside one JVM.
      - Separate `novel-crawl` instances do not share this lock.
3. <a name="synchronized-method"></a>`synchronized` method
   1. Function
      - `IdWorker.nextId`
   2. Lock
      - Lets only one thread generate an ID at a time through the same `IdWorker` object.
   3. Boundary
      - Separate JVM instances do not share this lock.
