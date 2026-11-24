# Novel Plus Flows

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> Traces complete user actions through the repository.

1. [Book Download](01-book-download.md): Streams one novel from MySQL to an administrator as a `.txt` response; Redis stores only a temporary lock for that book ID.
2. [Homepage Book-Setting Cache](02-homepage-book-setting-cache.md): Loads homepage book settings through Redis; a cache miss loads MySQL and caches the result for 24 hours.
3. [Comment Like Toggle](03-comment-like-toggle.md): Uses Redis as the only store for comment likes; the Set has no expiry.
4. [Registration CAPTCHA](04-registration-captcha.md): Stores the generated CAPTCHA text in Redis by client IP for five minutes, then compares it with the submitted CAPTCHA text during registration.
5. [AI Cover Generation](05-ai-cover-generation.md): The add-book request returns before the AI cover is ready. Redis temporarily passes the finished cover URL from the background task to the polling author page.
