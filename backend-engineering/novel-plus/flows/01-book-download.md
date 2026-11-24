# Book Download Flow

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> Streams one novel from MySQL to an administrator as a `.txt` response; Redis stores only a temporary lock for that book ID.

1. Request
   - The administrator clicks Download for a book.
2. Lock
   - `setIfAbsent(...)` checks and tries to create `bookIsDownloading:<bookId>` in Redis.
   - Returns `true`: Continue. Returns `false`: Reject the request.
3. Download
   - Read the chapters from MySQL and stream the `.txt` response; Redis does not store the book or file.
4. Finish
   - Delete the Redis key.
