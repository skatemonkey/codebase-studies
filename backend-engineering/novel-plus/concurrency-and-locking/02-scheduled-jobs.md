# Scheduled Jobs

[All studies](../../../README.md) · [Backend Engineering](../../README.md) · [novel-plus overview](../README.md)

> **Table of Contents**
>
> - [1. Creation and Configuration](#creation-and-configuration)
> - [2. Jobs](#jobs)
> - [3. Execution](#execution)

1. <a name="creation-and-configuration"></a>Creation and configuration
   1. Activation
      - `@EnableScheduling` registers a scheduling processor bean in each of the two application contexts: `novel-front` and `novel-crawl`.
   2. Scheduler beans and configuration

      | Application context | Scheduler bean | Configuration |
      | --- | --- | --- |
      | `novel-front` | **Custom** `taskScheduler`, declared by application code | `ThreadPoolTaskScheduler`, 5 threads |
      | `novel-crawl` | **Default** `taskScheduler`, registered by Spring Boot | `ThreadPoolTaskScheduler`, 1 thread |

   3. Jobs
      - Each job class is a Spring `@Service`, and `@Scheduled` marks its job method.
2. <a name="jobs"></a>Jobs

   | Job | Application | Trigger | Activation condition |
   | --- | --- | --- | --- |
   | `DailyIncomeStaSchedule.statistics` | `novel-front` | Every day at 00:00 | Always registered |
   | `MonthIncomeStaSchedule.statistics` | `novel-front` | First day of each month at 02:00 | Always registered |
   | `CrawlPicTransSchedule.trans` | `novel-front` | Every 10 minutes by `fixedRate` | `pic.save.type=2` |
   | `DiskMonitorSchedule.checkDiskUsage` | `novel-crawl` | Configured minutes after the previous run finishes | `disk-monitor.enabled=true` |

3. <a name="execution"></a>Execution
   1. Trigger styles
      - `cron`: Runs at a calendar time.
      - `fixedRate`: Measures the interval between task starts.
      - `fixedDelay`: Measures the interval from the previous completion.
   2. Runtime flow
      - Application starts → `@EnableScheduling` activates scheduling → Spring registers the `@Scheduled` methods → a trigger becomes due → the scheduler executes the method.
