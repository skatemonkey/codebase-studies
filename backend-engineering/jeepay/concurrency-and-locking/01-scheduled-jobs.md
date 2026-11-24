# Scheduled Jobs

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [Jeepay overview](../README.md)

> **Table of Contents**
>
> - [1. Creation and Configuration](#creation-and-configuration)
> - [2. Jobs](#jobs)
> - [3. Execution](#execution)

1. <a name="creation-and-configuration"></a>Creation and configuration
   1. Activation
      - `@EnableScheduling` registers the scheduling processor bean in the `jeepay-payment` application context.
   2. Scheduler beans and configuration

      | Application context | Scheduler bean | Configuration |
      | --- | --- | --- |
      | `jeepay-payment` | **Default** `taskScheduler`, registered by Spring Boot | `ThreadPoolTaskScheduler`, 1 thread |

   3. Jobs
      - Each job class is a Spring `@Component`, and `@Scheduled` marks its `start()` method.
2. <a name="jobs"></a>Jobs

   | Job | Application | Trigger | Activation condition |
   | --- | --- | --- | --- |
   | `PayOrderReissueTask.start` | `jeepay-payment` | Every minute | Always registered |
   | `RefundOrderReissueTask.start` | `jeepay-payment` | Every minute | Always registered |
   | `TransferOrderReissueTask.start` | `jeepay-payment` | Every minute | Always registered |
   | `PayOrderDivisionRecordReissueTask.start` | `jeepay-payment` | Every minute | Always registered |
   | `PayOrderExpiredTask.start` | `jeepay-payment` | Every minute | Always registered |
   | `RefundOrderExpiredTask.start` | `jeepay-payment` | Every minute | Always registered |

3. <a name="execution"></a>Execution
   1. Trigger
      - `cron = "0 0/1 * * * ?"`: Runs at second zero of every minute.
   2. Runtime flow
      - Application starts → `@EnableScheduling` activates scheduling → Spring registers the `@Scheduled` methods → a trigger becomes due → the scheduler executes `start()` on its single thread.
