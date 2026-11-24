# miaosha v2 User Purchase Flow

[All studies](../../README.md) · [Backend Engineering](../README.md) · [miaosha v2 overview](README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)
> - [1. Submit the Purchase Request](#1-submit-the-purchase-request)

## 0. Overview

> Traces the browser path from account access to the flash-sale result.

```text
Register (optional)
GET /registerv2 → POST /register
        ↓
Log in
GET /login/to_login → POST /login/loginin
        ↓
Browse products
GET /goods/to_list
        ↓
Open a product
GET /goods_detail.htm?goodsId={goodsId} → GET /goods/detail/{goodsId}
        ↓
Get the purchase CAPTCHA
GET /miaosha/verifyCode?goodsId={goodsId}
        ↓
Exchange the CAPTCHA answer for a temporary purchase path
GET /miaosha/path?goodsId={goodsId}&verifyCode={answer}
        ↓
Submit the purchase request
POST /miaosha/{path}/do_miaosha?goodsId={goodsId}
        ↓
Poll for the final result
GET /miaosha/result?goodsId={goodsId}
```

- Main purchase entry point: `POST /miaosha/{path}/do_miaosha?goodsId={goodsId}`.
- Login behavior: `/login/loginin` already creates the token cookie; `/login/create_token` is an alternative endpoint rather than another step in this flow.

1. [Submit the Purchase Request](#1-submit-the-purchase-request): Handles the purchase command from admission checks through queued order creation.

## 1. Submit the Purchase Request

> Handles the purchase command from admission checks through queued order creation.

1. Admission checks
   - API: `POST /miaosha/{path}/do_miaosha?goodsId={goodsId}`.
   - Authenticate and rate-limit the user.
   - Validate the temporary purchase path.
   - Reject a duplicate purchase or a product already marked as sold out.
2. Redis stock reservation
   - Atomically decrement the product stock in Redis.
   - Reject the request if no stock remains.
   - Otherwise, send `{user, goodsId}` to RabbitMQ and return the HTTP response immediately.
3. Asynchronous order creation
   - The RabbitMQ consumer receives the request.
   - It rechecks the stock and duplicate purchase.
   - It conditionally reduces MySQL stock through Dubbo.
   - It creates the order and caches the user-product order in Redis.
4. Result observation
   - The browser repeatedly calls `GET /miaosha/result?goodsId={goodsId}`.
   - `0`: Still processing.
   - Order ID: Purchase succeeded.
   - `-1`: Purchase failed.

- Key boundary: The purchase API does not create the order directly. It admits the request, reserves Redis stock, puts the work into RabbitMQ, and returns; the consumer creates the order afterward.
