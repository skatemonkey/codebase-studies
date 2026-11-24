# Database

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [12306 overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Tables](#1-tables)

## 0. Overview

> The MySQL database structure used by 12306.

- Deployment modes (run one at a time)

  | Mode | Java applications | Total databases | Sharding |
  | --- | --- | --- | --- |
  | Aggregate | `aggregation-service` (1 combined) | 1 (`12306`) | Table sharding |
  | Microservice | 4 microservices:<br>• `user-service` → 2 databases<br>• `ticket-service` → 1 database<br>• `order-service` → 2 databases<br>• `pay-service` → 2 databases | 7 | Database and table sharding |

- Shared server configuration
  - MySQL servers: 1 (`127.0.0.1:3306`).
  - Replicas: not configured.
  - Read/write splitting: not configured.

- [Tables](#1-tables): 20 logical tables shared by both deployment modes.
- [User Service Database Connection](01-user-service-database-connection.md): The database connection configuration used by `user-service`.
- [User Tables](02-user-tables.md): The relationships and columns of the four sharded user-service tables.

## 1. Tables

> 20 logical tables shared by both deployment modes.

- Total: 20 logical tables.
  - Sharded 🔀: 8.
  - Unsharded: 12.

- Why shard users but leave `t_ticket` unsharded?
  - Concern: one user can buy many tickets over time, so ticket rows can outnumber user rows—yet the user tables are already sharded.
  - Findings
    - The author’s reason remains unconfirmed.
    - Ticket history reads the **sharded order tables**.
    - No `t_ticket` archiving path was found in this version.

<table>
  <thead>
    <tr><th>Group</th><th>Table name</th><th>Purpose</th><th>Additional indexes</th></tr>
  </thead>
  <tbody>
    <tr><td rowspan="6">Users (6)</td><td><a href="02-user-tables.md#1-t_user"><code>t_user</code></a> 🔀</td><td>Account records.</td><td><code>UNIQUE (username, deletion_time)</code></td></tr>
    <tr><td><a href="02-user-tables.md#2-t_passenger"><code>t_passenger</code></a> 🔀</td><td>Passenger profiles belonging to an account.</td><td><code>INDEX (id_card)</code><br>Aggregate + microservice shards <code>_16</code>–<code>_31</code>:<br><code>INDEX (username)</code></td></tr>
    <tr><td><a href="02-user-tables.md#3-t_user_mail"><code>t_user_mail</code></a> 🔀</td><td>Email-to-username lookup.</td><td><code>UNIQUE (mail, deletion_time)</code></td></tr>
    <tr><td><a href="02-user-tables.md#4-t_user_phone"><code>t_user_phone</code></a> 🔀</td><td>Phone-to-username lookup.</td><td><code>UNIQUE (phone, deletion_time)</code></td></tr>
    <tr><td><code>t_user_deletion</code></td><td>Deleted-account identity history.</td><td>—</td></tr>
    <tr><td><code>t_user_reuse</code></td><td>Usernames available for reuse.</td><td><code>INDEX (username)</code></td></tr>
    <tr><td rowspan="9">Tickets (9)</td><td><code>t_region</code></td><td>Region and city metadata.</td><td>—</td></tr>
    <tr><td><code>t_station</code></td><td>Station codes, names, and regions.</td><td>—</td></tr>
    <tr><td><code>t_train</code></td><td>Train details and sale information.</td><td>—</td></tr>
    <tr><td><code>t_carriage</code></td><td>Train carriages and their capacity.</td><td><code>INDEX (train_id)</code></td></tr>
    <tr><td><code>t_train_station</code></td><td>Train stops and timetable information.</td><td><code>INDEX (train_id)</code></td></tr>
    <tr><td><code>t_train_station_relation</code></td><td>Purchasable departure/arrival pairs.</td><td><code>INDEX (train_id)</code></td></tr>
    <tr><td><code>t_train_station_price</code></td><td>Fares by route and seat type.</td><td><code>INDEX (train_id)</code></td></tr>
    <tr><td><code>t_seat</code></td><td>Seat availability for a train and station interval.</td><td><code>INDEX (train_id)</code></td></tr>
    <tr><td><code>t_ticket</code></td><td>Assigned passenger tickets and seats.</td><td>—</td></tr>
    <tr><td rowspan="3">Orders (3)</td><td><code>t_order</code> 🔀</td><td>Order headers and lifecycle state.</td><td><code>INDEX (user_id)</code><br><code>INDEX (order_sn)</code></td></tr>
    <tr><td><code>t_order_item</code> 🔀</td><td>Passenger and seat details within an order.</td><td><code>INDEX (order_sn)</code><br><code>INDEX (user_id)</code></td></tr>
    <tr><td><code>t_order_item_passenger</code> 🔀</td><td>Passenger identity-to-order lookup.</td><td><code>INDEX (id_card)</code></td></tr>
    <tr><td rowspan="2">Payments (2)</td><td><code>t_pay</code> 🔀</td><td>Payment transaction records.</td><td><code>UNIQUE (id)</code><br>Aggregate + microservice shards <code>_0</code>–<code>_15</code>:<br><code>INDEX (pay_sn)</code><br><code>INDEX (order_sn)</code></td></tr>
    <tr><td><code>t_refund</code></td><td>Refund records.</td><td><code>INDEX (user_id)</code><br><code>INDEX (order_sn)</code></td></tr>
  </tbody>
</table>
