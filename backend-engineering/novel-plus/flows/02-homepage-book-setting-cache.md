# Homepage Book-Setting Cache Flow

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> Loads homepage book settings through Redis; a cache miss loads MySQL and caches the result for 24 hours.

1. Request
   - The user opens the homepage.
   - `PageController.index` calls `BookServiceImpl.listBookSettingVO`.
2. Cache lookup
   - Read `indexBookSettingsKey:v2` from Redis.
   - Cache hit: Return the cached book settings.
3. Cache miss
   - Load the book settings through `BookSettingMapper.listVO`.
   - If MySQL returns no settings, initialize them first.
4. Cache write
   - Store the result in Redis as JSON with a 24-hour expiry.
   - Return the book settings to the homepage.
