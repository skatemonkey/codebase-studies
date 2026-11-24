# Flows

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [shortlink overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

<table>
  <thead>
    <tr><th>Area</th><th>Flow</th><th>Main path</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="3">Runtime</td><td>F01 · Aggregate startup</td><td>Load admin + project in one JVM.</td></tr>
    <tr><td>F02 · Microservice startup</td><td>Start gateway, admin, and project separately.</td></tr>
    <tr><td>F03 · Request authentication</td><td>Token → Redis session check → controller or rejection.</td></tr>
    <tr><td rowspan="12">Accounts and groups</td><td>F04 · Username availability</td><td>Username → Redis Bloom filter → availability.</td></tr>
    <tr><td>F05 · Registration</td><td>Save user → create default group.</td></tr>
    <tr><td>F06 · Login</td><td>Credentials → MySQL → Redis session → token.</td></tr>
    <tr><td>F07 · Session check</td><td>Token → Redis session check → login status.</td></tr>
    <tr><td>F08 · Logout</td><td>Delete the Redis session.</td></tr>
    <tr><td>F09 · View account</td><td>Username → MySQL → profile.</td></tr>
    <tr><td>F10 · Edit account</td><td>Verify owner → update profile.</td></tr>
    <tr><td>F11 · Create group</td><td>Check limit → generate ID → save group.</td></tr>
    <tr><td>F12 · List groups</td><td>Read groups → attach active-link counts.</td></tr>
    <tr><td>F13 · Rename group</td><td>Update the group name.</td></tr>
    <tr><td>F14 · Reorder groups</td><td>Update group positions.</td></tr>
    <tr><td>F15 · Delete group</td><td>Mark the group deleted.</td></tr>
    <tr><td rowspan="12">Short links</td><td>F16 · Create link</td><td>URL → short code → MySQL + Redis cache → short URL.</td></tr>
    <tr><td>F17 · Create with lock</td><td>Acquire Redis lock → create link → release lock.</td></tr>
    <tr><td>F18 · Batch creation</td><td>URLs → create links → Excel download.</td></tr>
    <tr><td>F19 · Fetch page title</td><td>URL → remote page → title.</td></tr>
    <tr><td>F20 · List links</td><td>Group → paginated links with today's counters.</td></tr>
    <tr><td>F21 · Edit link</td><td>Update link → invalidate affected Redis caches.</td></tr>
    <tr><td>F22 · Move between groups</td><td>Redis write lock → change group → update lookup.</td></tr>
    <tr><td>F23 · Visit link</td><td>Short URL → Redis cache → MySQL on miss → redirect or not-found.</td></tr>
    <tr><td>F24 · Recycle link</td><td>Disable link → clear Redis destination cache.</td></tr>
    <tr><td>F25 · Browse recycle bin</td><td>Query disabled links → paginated list.</td></tr>
    <tr><td>F26 · Restore link</td><td>Enable link → clear Redis not-found cache.</td></tr>
    <tr><td>F27 · Delete recycled link</td><td>Mark deleted → record deletion time.</td></tr>
    <tr><td rowspan="3">Statistics</td><td>F28 · Store visits</td><td>Visit → Redis Streams → consumer → MySQL.</td></tr>
    <tr><td>F29 · View statistics</td><td>Date range → link or group summary.</td></tr>
    <tr><td>F30 · View visit records</td><td>Link or group → paginated visit logs.</td></tr>
  </tbody>
</table>
