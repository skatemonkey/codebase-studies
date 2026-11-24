# Database Concurrency Control

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [Jeepay overview](../README.md)

> **Table of Contents**
>
> - [1. Expected-state SQL predicates](#expected-state-sql-predicates)
> - [2. SQL increments with amount predicates](#sql-increments-with-amount-predicates)
> - [3. Spring `@Transactional`](#spring-transactional)
> - [4. Database unique keys](#database-unique-keys)

1. <a name="expected-state-sql-predicates"></a>Expected-state SQL predicates
   1. Description
      - Updates the row only if its `state` still matches the value required by `WHERE`.
   2. Functions
      - Order states below use each entity's `STATE_*` constants.

      | Function | Expected state | Change |
      | --- | --- | --- |
      | [PayOrderService.updateInit2Ing](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderService.java#L58) | `INIT` | Sets `ING`. |
      | [PayOrderService.updateInit2Close](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderService.java#L99) | `INIT` | Sets `CLOSED`. |
      | [PayOrderService.updateIng2Success](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderService.java#L76) | `ING` | Sets `SUCCESS`. |
      | [PayOrderService.updateIng2Fail](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderService.java#L110) | `ING` | Sets `FAIL`. |
      | [PayOrderService.updateIng2Close](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderService.java#L89) | `ING` | Sets `CLOSED`. |
      | [PayOrderService.updateOrderExpired](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderService.java#L191) | `INIT` or `ING`; expired | Sets `CLOSED`. |
      | [RefundOrderService.updateInit2Ing](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/RefundOrderService.java#L62) | `INIT` | Sets `ING`. |
      | [RefundOrderService.updateIng2Success](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/RefundOrderService.java#L74) | `ING` | Sets `SUCCESS`; updates payment refund totals. |
      | [RefundOrderService.updateIng2Fail](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/RefundOrderService.java#L101) | `ING` | Sets `FAIL`. |
      | [RefundOrderService.updateOrderExpired](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/RefundOrderService.java#L130) | `INIT` or `ING`; expired | Sets `CLOSED`. |
      | [TransferOrderService.updateInit2Ing](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/TransferOrderService.java#L44) | `INIT` | Sets `ING`. |
      | [TransferOrderService.updateIng2Success](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/TransferOrderService.java#L57) | `ING` | Sets `SUCCESS`. |
      | [TransferOrderService.updateIng2Fail](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/TransferOrderService.java#L77) | `ING` | Sets `FAIL`. |
      | [PayOrderDivisionProcessService.processPayOrderDivision](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-payment/src/main/java/com/jeequan/jeepay/pay/service/PayOrderDivisionProcessService.java#L75) | Previously read `division_state`: `UNHAPPEN` or `WAIT_TASK` | Claims the order by setting `ING`; finishes only from `ING`. |
      | [PayOrderDivisionRecordService.updateResendState](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderDivisionRecordService.java#L62) | Order `division_state = FINISH`; records `state = FAIL` | Sets order `WAIT_TASK`; records `WAIT`. |
      | [PayOrderDivisionRecordService.updateRecordSuccessOrFail](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderDivisionRecordService.java#L43) | Records `state = WAIT` | Sets the channel result state. |
      | [PayOrderDivisionRecordService.updateRecordSuccessOrFailBySingleItem](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderDivisionRecordService.java#L32) | Record `state = ACCEPT` | Sets the channel result state. |

      - `updateIng2SuccessOrFail` in each order service delegates to its guarded success or failure method.
   3. Examples
      - Payment example
        - State-update excerpt from `PayOrderService.updateIng2Success`:

          ```java
          PayOrder updateRecord = new PayOrder();
          updateRecord.setState(PayOrder.STATE_SUCCESS);

          return update(updateRecord, new LambdaUpdateWrapper<PayOrder>()
                  .eq(PayOrder::getPayOrderId, payOrderId)
                  .eq(PayOrder::getState, PayOrder.STATE_ING));
          ```

        - Equivalent state update, with other updated fields omitted:

          ```sql
          UPDATE t_pay_order SET state = 2
          WHERE pay_order_id = ? AND state = 1;
          ```

        - `1`: Paying. `2`: Successful.
        - Two instances target the same paying order. A updates and commits; B then finds that `state = 1` no longer matches and updates zero rows.
        - The guarded service update returns `false` when no row matches.
      - Profit-sharing example
        - `processPayOrderDivision` checks the previously read state again in the update:

          ```java
          boolean updPayOrder = payOrderService.update(new LambdaUpdateWrapper<PayOrder>()
                  .set(PayOrder::getDivisionState, PayOrder.DIVISION_STATE_ING)
                  .eq(PayOrder::getPayOrderId, payOrderId)
                  .eq(PayOrder::getDivisionState, payOrder.getDivisionState()));
          ```

        - If another worker has already claimed the order, the update fails and the method throws `BizException` before calling the channel.

2. <a name="sql-increments-with-amount-predicates"></a>SQL increments with amount predicates
   1. Description
      - Adds `currentRefundAmount` to `refund_amount` only when the result is at most `amount`.
      - `currentRefundAmount`: Refund amount passed from the refund record.
      - `refund_amount`: Refund total already stored in the payment row.
      - `amount`: Original payment amount stored in that row.
   2. Function
      - [PayOrderMapper.updateRefundAmountAndCount](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/mapper/PayOrderMapper.xml#L86)
   3. SQL
      - Mapper SQL; XML `&lt;=` is shown as `<=`:

        ```sql
        UPDATE t_pay_order
        SET refund_times = refund_times + 1,
            refund_state = CASE
                WHEN refund_amount + #{currentRefundAmount} >= amount THEN 2
                ELSE 1
            END,
            state = CASE WHEN refund_state = 2 THEN 5 ELSE 2 END,
            refund_amount = refund_amount + #{currentRefundAmount}
        WHERE pay_order_id = #{payOrderId}
          AND state = 2
          AND refund_amount + #{currentRefundAmount} <= amount
          AND refund_state IN (0, 1);
        ```

      - Requires a successful payment (`state = 2`) with no refund or a partial refund (`refund_state IN (0, 1)`).
      - Also increments `refund_times` and updates `refund_state` and `state`.
      - `refund_state` uses the old `refund_amount`; `state` uses the new `refund_state`; increment `refund_amount` last.
   4. Concurrent example
      - `amount = 100`, `refund_amount = 70`; two different refunds each have `currentRefundAmount = 20`.
      - A adds 20: `refund_amount = 90`.
      - B checks `90 + 20 <= 100`: False; zero rows update.
      - The limit check and increment happen in the same SQL statement.

3. <a name="spring-transactional"></a>Spring `@Transactional`
   1. Rule
      - Commits related database changes together or rolls them back.
      - State and amount predicates still decide whether each update is allowed.
   2. Functions
      - [RefundOrderService.updateIng2Success](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/RefundOrderService.java#L74)
        1. Changes the refund from `ING` to `SUCCESS`.
        2. Passes the refund record's amount to `updateRefundAmountAndCount`.
        3. If the payment-total update affects zero rows, throws `BizException`; the refund-state change rolls back.
      - [PayOrderDivisionRecordService.updateResendState](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/PayOrderDivisionRecordService.java#L62)
        1. Changes the order's division state from `FINISH` to `WAIT_TASK`.
        2. Changes failed division records to `WAIT` and generates a new batch ID.
        3. Throws `BizException` if either update changes no rows; both changes roll back.
      - [RefundOrderService](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/RefundOrderService.java#L100)
        - `updateIng2Fail` and `updateIng2SuccessOrFail` also use `@Transactional`.
      - [TransferOrderService](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/jeepay-service/src/main/java/com/jeequan/jeepay/service/impl/TransferOrderService.java#L56)
        - `updateIng2Success`, `updateIng2Fail`, and `updateIng2SuccessOrFail` use `@Transactional` around the guarded transfer-state update.
   3. Refund example
      - Excerpt from `RefundOrderService.updateIng2Success`, inside its `@Transactional` method:

        ```java
        if (!update(updateRecord, new LambdaUpdateWrapper<RefundOrder>()
                .eq(RefundOrder::getRefundOrderId, refundOrderId)
                .eq(RefundOrder::getState, RefundOrder.STATE_ING))) {
            return false;
        }

        RefundOrder refundOrder = getOne(RefundOrder.gw()
                .select(RefundOrder::getPayOrderId, RefundOrder::getRefundAmount)
                .eq(RefundOrder::getRefundOrderId, refundOrderId));

        int updateCount = payOrderMapper.updateRefundAmountAndCount(
                refundOrder.getPayOrderId(), refundOrder.getRefundAmount());
        if (updateCount <= 0) {
            throw new BizException("更新订单数据异常");
        }
        ```

      - `BizException` extends `RuntimeException`, so this exception triggers Spring's default rollback.

4. <a name="database-unique-keys"></a>Database unique keys
   1. Rule
      - Rejects a second record with the same unique-column combination.
      - Two instances attempting the same insert cannot both create that record.
   2. Tables

      | Table | Unique columns | Duplicate rejected |
      | --- | --- | --- |
      | [t_pay_order](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/docs/sql/init.sql#L289) | `mch_no`, `mch_order_no` | Same merchant payment order number. |
      | [t_refund_order](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/docs/sql/init.sql#L367) | `mch_no`, `mch_refund_no` | Same merchant refund number. |
      | [t_transfer_order](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/docs/sql/init.sql#L402) | `mch_no`, `mch_order_no` | Same merchant transfer order number. |
      | [t_mch_notify_record](https://github.com/jeequan/jeepay/blob/e1ac9086d679826bab593f20431b7ecd292eb685/docs/sql/init.sql#L313) | `order_id`, `order_type` | Same order and notification type. |

   3. Definition example
      - `t_pay_order` defines:

        ```sql
        UNIQUE KEY `Uni_MchNo_MchOrderNo` (`mch_no`, `mch_order_no`)
        ```

      - Merchant A can reuse a number used by merchant B. The same merchant cannot insert that payment order number twice.
