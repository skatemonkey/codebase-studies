# Redis Integration

[All studies](../../README.md) · [Backend Engineering](../README.md) · [novel-plus overview](README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Configuration](#1-configuration)
> - [2. Usage](#2-usage)

## 0. Overview

1. [Configuration](#1-configuration)
2. [Usage](#2-usage)
   - Native value types
     - String
     - Set

## 1. Configuration

> This repository only defines Redis connection settings.<br>It does not define Redis server-side configuration.

1. Connection settings
   - `novel-front` and `novel-crawl`
     - Development: `novel-common/src/main/resources/application-common-dev.yml`
     - Production: `novel-common/src/main/resources/application-common-prod.yml`
   - `novel-admin`
     - Development: `application-dev.yml`
     - Production: `application-prod.yml`
   - Values
     - Host, port, password, and connection timeout.
     - Checked-in address: `127.0.0.1:6379`.

2. Server settings: Not defined in this repository.

## 2. Usage

<table>
  <thead>
    <tr>
      <th>Module</th>
      <th>Key</th>
      <th>Value</th>
      <th>Purpose</th>
      <th>Expiry</th>
      <th>Used in</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td rowspan="13"><code>novel-front</code></td>
      <td><code>indexBookSettingsKey:v2</code></td>
      <td>Homepage book-setting records<br><code>String · JSON</code></td>
      <td>Avoids repeatedly loading homepage recommendation settings from MySQL.</td>
      <td>24 hours; deleted after administration updates or manual cache refresh.</td>
      <td><a href="flows/02-homepage-book-setting-cache.md"><code>BookServiceImpl</code></a>, <code>BookSettingController</code>, <code>CacheController</code></td>
    </tr>
    <tr>
      <td><code>indexClickBankBookKey</code></td>
      <td>Click-ranking books<br><code>String · JSON</code></td>
      <td>Avoids repeatedly calculating the click ranking.</td>
      <td>5,000 seconds.</td>
      <td><code>BookServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>indexNewBookKey</code></td>
      <td>New-book ranking<br><code>String · JSON</code></td>
      <td>Avoids repeatedly calculating the new-book ranking.</td>
      <td>1 hour.</td>
      <td><code>BookServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>indexUpdateBookKey</code></td>
      <td>Recently updated books<br><code>String · JSON</code></td>
      <td>Avoids repeatedly loading the recent-update list.</td>
      <td>10 minutes.</td>
      <td><code>BookServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>indexNewsKey</code></td>
      <td>Homepage news<br><code>String · JSON</code></td>
      <td>Avoids repeatedly loading homepage news from MySQL.</td>
      <td>12 hours; deleted after administration changes or manual cache refresh.</td>
      <td><code>NewsServiceImpl</code>, <code>NewsController</code>, <code>CacheController</code></td>
    </tr>
    <tr>
      <td><code>indexLinkKey</code></td>
      <td>Homepage friend links<br><code>String · JSON</code></td>
      <td>Avoids repeatedly loading friend links from MySQL.</td>
      <td>24 hours; deleted after administration changes or manual cache refresh.</td>
      <td><code>FriendLinkServiceImpl</code>, <code>FriendLinkController</code>, <code>CacheController</code></td>
    </tr>
    <tr>
      <td><code>randomValidateCodeKey:&lt;IP&gt;</code></td>
      <td>Registration CAPTCHA text<br><code>String</code></td>
      <td>Validates a registration request against the CAPTCHA issued to its IP address.</td>
      <td>5 minutes.</td>
      <td><a href="flows/04-registration-captcha.md"><code>FileController</code></a>, <a href="flows/04-registration-captcha.md"><code>UserController</code></a></td>
    </tr>
    <tr>
      <td><code>templateDirKey&lt;clientId&gt;</code></td>
      <td><code>mobile/</code> or an empty template prefix<br><code>String</code></td>
      <td>Remembers whether a browser should receive the mobile or desktop templates.</td>
      <td>24 hours.</td>
      <td><code>NovelFilter</code>, <code>ThreadLocalUtil</code></td>
    </tr>
    <tr>
      <td><code>aiGenPic&lt;bookId&gt;</code></td>
      <td>Generated cover-image URL<br><code>String</code></td>
      <td>Lets the author interface retrieve an asynchronously generated AI cover.</td>
      <td>1 hour.</td>
      <td><a href="flows/05-ai-cover-generation.md"><code>BookServiceImpl</code></a></td>
    </tr>
    <tr>
      <td><code>like:comment:&lt;commentId&gt;</code></td>
      <td>User IDs<br><code>Set</code></td>
      <td>Tracks comment likes and derives the current like count.</td>
      <td>No expiry is set.</td>
      <td><a href="flows/03-comment-like-toggle.md"><code>LikeServiceImpl</code></a></td>
    </tr>
    <tr>
      <td><code>unlike:comment:&lt;commentId&gt;</code></td>
      <td>User IDs<br><code>Set</code></td>
      <td>Tracks comment dislikes and derives the current dislike count.</td>
      <td>No expiry is set.</td>
      <td><a href="flows/03-comment-like-toggle.md"><code>LikeServiceImpl</code></a></td>
    </tr>
    <tr>
      <td><code>like:comment:reply:&lt;replyId&gt;</code></td>
      <td>User IDs<br><code>Set</code></td>
      <td>Tracks reply likes and derives the current like count.</td>
      <td>No expiry is set.</td>
      <td><a href="flows/03-comment-like-toggle.md"><code>LikeServiceImpl</code></a></td>
    </tr>
    <tr>
      <td><code>unlike:comment:reply:&lt;replyId&gt;</code></td>
      <td>User IDs<br><code>Set</code></td>
      <td>Tracks reply dislikes and derives the current dislike count.</td>
      <td>No expiry is set.</td>
      <td><a href="flows/03-comment-like-toggle.md"><code>LikeServiceImpl</code></a></td>
    </tr>
    <tr>
      <td rowspan="4"><code>novel-admin</code></td>
      <td><code>shiro_redis_session:&lt;sessionId&gt;</code></td>
      <td>Serialized Shiro administration session<br><code>String · binary serialization</code></td>
      <td>Shares and manages logged-in administration sessions.</td>
      <td>1,800 seconds.</td>
      <td><code>ShiroConfig</code>, <code>RedisSessionDAO</code>, <code>SessionServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>shiro_redis_cache:*</code></td>
      <td>Serialized Shiro security cache entry<br><code>String · binary serialization</code></td>
      <td>Caches administration authentication and authorization data.</td>
      <td>1,800 seconds.</td>
      <td><code>ShiroConfig</code>, <code>RedisCacheManager</code>, <code>RedisCache</code></td>
    </tr>
    <tr>
      <td>Spring-generated cache key</td>
      <td>Administration menu tree for a user<br><code>String · Spring serialization</code></td>
      <td>Caches <code>getSysMenuTree</code> through Spring Cache.</td>
      <td>No explicit cache name or expiry appears at the method.</td>
      <td><code>MenuServiceImpl</code></td>
    </tr>
    <tr>
      <td><code>bookIsDownloading:&lt;bookId&gt;</code></td>
      <td><code>1</code><br><code>String</code></td>
      <td>Stops two requests from downloading the same book at the same time.</td>
      <td>10 minutes at most; deleted when the download finishes.</td>
      <td><a href="flows/01-book-download.md"><code>BookController.download</code></a></td>
    </tr>
    <tr>
      <td rowspan="2"><code>novel-crawl</code></td>
      <td><code>testParse&lt;url&gt;</code></td>
      <td>Downloaded source-page HTML<br><code>String</code></td>
      <td>Reuses source HTML while testing a crawler rule.</td>
      <td>10 minutes.</td>
      <td><code>CrawlController</code></td>
    </tr>
    <tr>
      <td><code>crawlSource:chapterCount:&lt;sourceId&gt;</code></td>
      <td>Number of chapters collected from one crawler source<br><code>String · integer</code></td>
      <td>Accumulates crawler-source collection statistics.</td>
      <td>No expiry is set.</td>
      <td><code>CrawlParser</code></td>
    </tr>
  </tbody>
</table>
