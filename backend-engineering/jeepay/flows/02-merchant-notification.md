# Merchant Notification Flow

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [Jeepay overview](../README.md)

> **Table of Contents**
>
> - [0. Overview](#0-overview)

## 0. Overview

> Checks for an existing notification record before inserting one and queuing the merchant callback.

1. Trigger
   - After payment succeeds, `PayOrderProcessService.confirmSuccess` calls `PayMchNotifyService.payOrderNotify`.
   - If `notifyUrl` is empty, return.
2. Java duplicate check
   - [PayMchNotifyService.payOrderNotify](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-payment/src/main/java/com/jeequan/jeepay/pay/service/PayMchNotifyService.java#L62) calls `mchNotifyRecordService.findByPayOrder(payOrderId)`.
   - The query matches `order_id` and `order_type = TYPE_PAY_ORDER`.
   - Existing record: Return before inserting or publishing another message.
3. Database insert
   - Build the signed callback URL and insert a `MchNotifyRecord` into `t_mch_notify_record`.
   - [Uni_OrderId_Type](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/docs/sql/init.sql#L313) rejects a duplicate `(order_id, order_type)` even if concurrent calls both passed the Java check.
   - The insert's `catch (Exception)` returns before publishing to MQ.
4. MQ message
   - After insertion succeeds, send `PayOrderMchNotifyMQ` containing the new `notifyId`.
5. Merchant callback
   - [PayOrderMchNotifyMQReceiver.receive](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-payment/src/main/java/com/jeequan/jeepay/pay/mq/PayOrderMchNotifyMQReceiver.java#L53) loads the notification record.
   - Stop if the record is missing, its state is not `STATE_ING`, or its attempt limit is reached.
   - Send an HTTP `POST` to the merchant's callback URL.
6. Result
   - Response `SUCCESS`: Mark the notification successful.
   - Other response or HTTP failure: Queue a delayed retry, or mark failure when the attempt limit is reached.
   - Retries reuse the same `notifyId` and notification record.
