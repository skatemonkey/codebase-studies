# Payment Order Creation Flow

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [Jeepay overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> Creates a payment order with a Java duplicate check and a database unique key.

1. Request
   - The merchant calls `POST /api/pay/unifiedOrder` in `jeepay-payment`.
   - `UnifiedOrderController.unifiedOrder` validates the request and merchant signature, then calls `AbstractPayOrderController.unifiedOrder`.
2. Java duplicate check
   - [AbstractPayOrderController.unifiedOrder](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-payment/src/main/java/com/jeequan/jeepay/pay/ctrl/payorder/AbstractPayOrderController.java#L117) counts payment orders with the same `mch_no` and `mch_order_no`.
   - Existing order: Throw `BizException` and return an error response.
   - This check runs only when creating a new order.
3. Database insert
   - Validate the merchant's payment configuration and create a `PayOrder` in `STATE_INIT`.
   - `payOrderService.save(payOrder)` inserts it into `t_pay_order`.
   - [Uni_MchNo_MchOrderNo](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/docs/sql/init.sql#L289) rejects a duplicate `(mch_no, mch_order_no)` even if concurrent requests both passed the Java check.
   - An insert exception returns an error response before calling the payment channel.
4. Payment channel
   - `paymentService.pay(...)` calls the selected payment channel.
   - For `QR_CASHIER`, return the cashier URL or QR-code URL after insertion; channel payment happens later.
5. Result
   - `processChannelMsg(...)` updates the order according to the channel result.
   - Confirmed success calls `PayOrderProcessService.confirmSuccess`, which starts merchant notification.
   - Return the signed payment response to the merchant.
