# Comment Like Toggle Flow

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> Uses Redis as the only store for comment likes; the Set has no expiry.

1. Request
   - The user calls `POST /book/toggleCommentLike` with a comment ID.
2. Set selection
   - `LikeServiceImpl` uses `like:comment:<commentId>`.
3. Atomic toggle
   - A Redis Lua script checks whether the Set contains the user ID.
   - Present: Remove it with `SREM`.
   - Absent: Add it with `SADD`.
4. Response
   - `SCARD` returns the updated number of likes.
