# Custom Thread Pool Task Executor

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [Jeepay overview](../README.md)

> **Table of Contents**
>
> - [1. Creation and Configuration](#creation-and-configuration)
> - [2. Tasks](#tasks)
> - [3. Execution](#execution)

1. <a name="creation-and-configuration"></a>Creation and configuration
   1. Activation
      - `@EnableAsync` in `MqThreadExecutor` registers an async annotation processor bean in each application context.
   2. Creation
      - `MqThreadExecutor.mqQueue4PayOrderMchNotifyExecutor()` in `jeepay-components-mq` creates a custom `ThreadPoolTaskExecutor` bean named `mqQueue4PayOrderMchNotifyExecutor`.
      - Each of `jeepay-payment`, `jeepay-manager`, and `jeepay-merchant` registers its own bean in its application context.
      - Spring's `ThreadPoolTaskExecutor` uses a JDK `ThreadPoolExecutor` internally.
   3. Configuration
      - Core threads: 20
      - Maximum threads: 300
      - Keep-alive time: 60 seconds
      - Queue: `LinkedBlockingQueue`, capacity 10
      - Rejection policy: `CallerRunsPolicy`
      - Thread name prefix: `payOrderMchNotifyExecutor-`
2. <a name="tasks"></a>Tasks
   - Both tasks run in `jeepay-payment` and share its executor bean.
   - `PayOrderMchNotifyMQReceiver.receive`: Delivers merchant callbacks and retries failures.
   - `PayOrderReissueMQReceiver.receive`: Rechecks pending payment results.
3. <a name="execution"></a>Execution
   1. Async methods
      - `@Async` is on the RocketMQ adapter methods in `jeepay-components-mq`:
        - `PayOrderMchNotifyRocketMQReceiver.onMessage`
        - `PayOrderReissueRocketMQReceiver.onMessage`

      ```java
      @Async(MqThreadExecutor.EXECUTOR_PAYORDER_MCH_NOTIFY)
      public void onMessage(String message) {
          this.receiveMsg(message);
      }
      ```

      - `EXECUTOR_PAYORDER_MCH_NOTIFY` resolves to `mqQueue4PayOrderMchNotifyExecutor`.
      - `receiveMsg()` parses the message and calls the business receiver's `receive(...)`.
   2. Runtime flow
      - RocketMQ consumer thread → `@Async` submits the task → custom pool worker → business receiver's `receive(...)`.
      - Normally, the consumer thread returns after submission without waiting for business work to finish.
   3. Pool behavior
      - Worker threads are created as tasks arrive, up to the core size of 20.
      - Further tasks queue; when the 10-task queue fills, the pool can grow to 300 threads.
      - When the running pool and queue are full, `CallerRunsPolicy` executes the task on the submitting RocketMQ consumer thread.
      - Threads above the core size stop after 60 seconds idle.
