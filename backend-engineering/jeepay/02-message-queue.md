# Message Queue (MQ)

[All studies](../../README.md) · [Backend Engineering](../README.md) · [Jeepay overview](README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

- **MQ:** `RocketMQ`.
- **Coverage:** 6 topics and 8 consumers, assuming one instance of each application.
- **Application totals:** `jeepay-payment`: 5; `jeepay-merchant`: 2; `jeepay-manager`: 1.
- **Message queues:** Per-topic counts come from the broker; Jeepay does not specify them.
- **Consumers:** One per topic per application instance; thread counts are separate.
- **Consumption:** `QUEUE_` shares work within a consumer group; `BROADCAST_` delivers to every subscribed consumer.

<table>
  <thead>
    <tr><th>Topic</th><th>Message queues</th><th>Consuming application</th><th>Consumers per application instance</th></tr>
  </thead>
  <tbody>
    <tr>
      <td><code>QUEUE_PAY_ORDER_MCH_NOTIFY</code></td>
      <td rowspan="8">Not specified</td>
      <td><code>jeepay-payment</code></td>
      <td>1</td>
    </tr>
    <tr>
      <td><code>QUEUE_PAY_ORDER_REISSUE</code></td>
      <td><code>jeepay-payment</code></td>
      <td>1</td>
    </tr>
    <tr>
      <td><code>QUEUE_PAY_ORDER_DIVISION</code></td>
      <td><code>jeepay-payment</code></td>
      <td>1</td>
    </tr>
    <tr>
      <td><code>BROADCAST_RESET_ISV_MCH_APP_INFO_CONFIG</code></td>
      <td><code>jeepay-payment</code></td>
      <td>1</td>
    </tr>
    <tr>
      <td><code>QUEUE_CLEAN_MCH_LOGIN_AUTH_CACHE</code></td>
      <td><code>jeepay-merchant</code></td>
      <td>1</td>
    </tr>
    <tr>
      <td rowspan="3"><code>BROADCAST_RESET_APP_CONFIG</code></td>
      <td><code>jeepay-payment</code></td>
      <td>1</td>
    </tr>
    <tr>
      <td><code>jeepay-merchant</code></td>
      <td>1</td>
    </tr>
    <tr>
      <td><code>jeepay-manager</code></td>
      <td>1</td>
    </tr>
  </tbody>
</table>
