# AI Cover Generation Flow

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> The add-book request returns before the AI cover is ready. Redis temporarily passes the finished cover URL from the background task to the polling author page.

1. Add book
   - The author calls `POST /author/addBook` without a local cover.
   - `BookServiceImpl.addBook` saves the new book to MySQL.
   - It submits cover generation as a background `Runnable` and returns without waiting for the cover.
2. Continue on two independent paths
   - Background task
     - Generate the cover through the configured AI image model.
     - Download the image to local storage.
     - Save the permanent cover URL in MySQL.
     - Save the same URL as `aiGenPic<bookId>` in Redis for one hour.
   - Author page
     - When the first book on the first page has `/images/default.gif`, the page starts polling for that book.
     - Call `GET /author/queryAiGenPic?bookId=<bookId>` every three seconds for up to ten seconds.
     - Each request reads only `aiGenPic<bookId>` from Redis.
3. Read the temporary result
   - Redis key does not exist
     - The background task has not published the URL yet.
     - Return no URL and continue polling.
   - Redis key exists
     - Return the generated cover URL.
     - Replace the default cover and stop polling.
4. Keep the permanent result
   - The image remains in local storage, and its URL remains in MySQL.
   - The temporary Redis key expires after one hour.
