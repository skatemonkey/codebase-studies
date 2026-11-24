# miaosha v2 Spike Traffic Strategy

[All studies](../../README.md) · [Backend Engineering](../README.md) · [miaosha v2 overview](README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> miaosha v2 protects MySQL from a flash-sale traffic spike by rejecting unnecessary requests, reserving limited stock in Redis, and buffering admitted work in RabbitMQ before MySQL performs the final stock update and order write.

1. Reject unnecessary traffic early.
   - Purchase CAPTCHA API: `GET /miaosha/verifyCode?goodsId={goodsId}`.
   - Temporary purchase path API: `GET /miaosha/path?goodsId={goodsId}&verifyCode={answer}`.
2. Reserve scarce stock in Redis.
3. Buffer order creation with RabbitMQ.
4. Let MySQL make the final decision.
