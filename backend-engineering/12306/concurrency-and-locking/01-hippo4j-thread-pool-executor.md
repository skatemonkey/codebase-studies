# Hippo4j Thread Pool Executor

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [12306 overview](../README.md)

> **Table of Contents**
>
> - [1. Thread Pool Setup](#thread-pool-setup)
> - [2. Thread Pool Usage](#thread-pool-usage)
> - [3. Task Submission Methods](#task-submission-methods)
>   - [3.1 `Callable` through `submit(...)`](#callable)

1. <a name="thread-pool-setup"></a>Thread Pool Setup
   1. Creation
      - `Hippo4jThreadPoolConfiguration.selectSeatThreadPoolExecutor()` creates one `ThreadPoolExecutor` bean in the application context; its consumers submit tasks to that bean.
      - `@DynamicThreadPool` enables Hippo4j management.
      - Source file: `services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/config/Hippo4jThreadPoolConfiguration.java`
   2. Configuration
      - Source files:
        - `services/ticket-service/src/main/java/org/opengoofy/index12306/biz/ticketservice/config/Hippo4jThreadPoolConfiguration.java`
        - `services/ticket-service/src/main/resources/application.yaml`
        - `services/aggregation-service/src/main/resources/application.yaml`
      - Pool ID: `select-seat-thread-pool-executor`.
        | Setting | Java defaults | Hippo4j YAML |
        | --- | --- | --- |
        | Core threads | 24 | 20 |
        | Maximum threads | 36 | 40 |
        | Keep-alive time | 60 minutes | 9,999 seconds |
      - Queue: `SynchronousQueue` — direct handoff; capacity 0.
      - Rejection policy: `CallerRunsPolicy` — caller runs tasks when saturated.
      - Core thread timeout: enabled.
      - Hippo4j applies configuration by pool ID; Nacos can refresh it.
2. <a name="thread-pool-usage"></a>Thread Pool Usage
   - `TrainSeatTypeSelector`: Runs parallel seat-selection tasks.

   > - User choices:
   >   - Seat class: fixed; insufficient seats → booking fails.
   >   - Window/aisle preference: optional, not guaranteed.
   > - System assigns the actual carriage and seat number.
3. <a name="task-submission-methods"></a>Task Submission Methods
   1. <a name="callable"></a>`Callable` through `submit(...)`
      1. Submission
         - Multiple seat types: one task per type.
         - One seat type: runs inline.

         ```java
         Future<List<TrainPurchaseTicketRespDTO>> completableFuture =
                 selectSeatThreadPoolExecutor.submit(() ->
                         distributeSeats(trainType, seatType, requestParam, passengerSeatDetails));
         ```

         - The task returns allocated seats for one seat type.
         - The variable `completableFuture` has type `Future`.
      2. <a name="future-result"></a>Result

         ```java
         List<TrainPurchaseTicketRespDTO> seats = completableFuture.get();
         ```

         - `get()` waits for a task and returns its seat list.
         - Combine all returned seat lists into one list for the booking.
