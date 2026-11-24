# User Tables

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [12306 overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. t_user](#1-t_user)
> - [2. t_passenger](#2-t_passenger)
> - [3. t_user_mail](#3-t_user_mail)
> - [4. t_user_phone](#4-t_user_phone)

## 0. Overview

> The relationships and columns of the four sharded user-service tables.

- Tables and columns
  1. [t_user](#1-t_user) (18 columns).
  2. [t_passenger](#2-t_passenger) (12 columns).
  3. [t_user_mail](#3-t_user_mail) (7 columns).
  4. [t_user_phone](#4-t_user_phone) (7 columns).

- Table relationships
  - Account (`t_user`): stores account details, including `mail` and `phone`.
  - Login lookup (`t_user_mail`, `t_user_phone`): finds the username for email/phone login.
    - Flow
      - Email login: find `username` in `t_user_mail`.
      - Phone login: find `username` in `t_user_phone`.
      - Then query `t_user` by username to check the password.
      - Logout: delete the cached token; no table lookup.
    - Why
      - Email and phone already exist in `t_user`, but its shard is chosen by `username`. Searching it directly by email/phone would require querying every user shard.
      - The lookup tables are sharded by `mail` or `phone`, so the first query reaches one lookup shard. The returned username then routes the account query to one user shard.
      - Queries across all user shards
        - Current project: no active query across all `t_user` shards was found.
          - There is no admin user-management page or API for listing all users.
          - `queryUserByUserId()` would query all shards because it searches by `id` without `username`, but no caller or exposed API endpoint was found.
        - Reminder: what if an admin user-management page is needed?
          - Listing or filtering users without `username` can require querying all shards.
  - Saved passengers (`t_passenger`): one account can save multiple passengers, linked by `username`.

## 1. t_user

| Column | Type | Meaning |
| --- | --- | --- |
| `id` | `bigint(20) unsigned` | Row ID (primary key). |
| `username` | `varchar(256)` | Account username. |
| `password` | `varchar(512)` | Account password. |
| `real_name` | `varchar(256)` | Real name. |
| `region` | `varchar(64)` | Country or region. |
| `id_type` | `int(3)` | Identity document type. |
| `id_card` | `varchar(256)` | Identity document number. |
| `phone` | `varchar(128)` | Mobile phone number. |
| `telephone` | `varchar(128)` | Landline number. |
| `mail` | `varchar(256)` | Email address. |
| `user_type` | `int(3)` | Passenger category. |
| `verify_status` | `int(3)` | Verification status. |
| `post_code` | `varchar(64)` | Postal code. |
| `address` | `varchar(1024)` | Address. |
| `deletion_time` | `bigint(20)` | Account deletion timestamp. |
| `create_time` | `datetime` | Record creation time. |
| `update_time` | `datetime` | Last update time. |
| `del_flag` | `tinyint(1)` | Logical deletion flag. |

## 2. t_passenger

| Column | Type | Meaning |
| --- | --- | --- |
| `id` | `bigint(20) unsigned` | Row ID (primary key). |
| `username` | `varchar(256)` | Username of the account that saved this passenger. |
| `real_name` | `varchar(128)` | Passenger's real name. |
| `id_type` | `int(3)` | Identity document type. |
| `id_card` | `varchar(256)` | Identity document number. |
| `discount_type` | `int(3)` | Discount category. |
| `phone` | `varchar(128)` | Passenger's mobile phone number. |
| `create_date` | `datetime` | Date the passenger was added. |
| `verify_status` | `int(3)` | Verification status. |
| `create_time` | `datetime` | Record creation time. |
| `update_time` | `datetime` | Last update time. |
| `del_flag` | `tinyint(1)` | Logical deletion flag. |

## 3. t_user_mail

| Column | Type | Meaning |
| --- | --- | --- |
| `id` | `bigint(20)` | Row ID (primary key). |
| `username` | `varchar(256)` | Username associated with this email. |
| `mail` | `varchar(256)` | Email used to find the username. |
| `deletion_time` | `bigint(20)` | Deletion timestamp. |
| `create_time` | `datetime` | Record creation time. |
| `update_time` | `datetime` | Last update time. |
| `del_flag` | `tinyint(1)` | Logical deletion flag. |

## 4. t_user_phone

| Column | Type | Meaning |
| --- | --- | --- |
| `id` | `bigint(20)` | Row ID (primary key). |
| `username` | `varchar(256)` | Username associated with this phone number. |
| `phone` | `varchar(128)` | Phone number used to find the username. |
| `deletion_time` | `bigint(20)` | Deletion timestamp. |
| `create_time` | `datetime` | Record creation time. |
| `update_time` | `datetime` | Last update time. |
| `del_flag` | `tinyint(1)` | Logical deletion flag. |
