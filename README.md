# Codebase Studies

Source-code studies by [Lok Shang Ru](https://github.com/skatemonkey), focused on how backend systems work and how their design choices affect behavior.

The notes connect system architecture to concrete implementation. They include source references, observed guarantees, and questions that still need verification.

## Start here

| Study | Question explored | Read |
| --- | --- | --- |
| Jeepay | How do payment operations coordinate state changes and prevent duplicate records? | [Database concurrency control](backend-engineering/jeepay/concurrency-and-locking/03-database-concurrency-control.md) · [Merchant notifications](backend-engineering/jeepay/flows/02-merchant-notification.md) |
| 12306 | How does ticket purchasing combine parallel work with shared inventory coordination? | [Concurrency and locking](backend-engineering/12306/concurrency-and-locking/README.md) · [Hippo4j thread pool](backend-engineering/12306/concurrency-and-locking/01-hippo4j-thread-pool-executor.md) |
| novel-plus | How are request-time work, background tasks, and shared-state locking organized? | [Concurrency and locking](backend-engineering/novel-plus/concurrency-and-locking/README.md) |
| Backtesting.py | How does a strategy interact with data and the simulation engine? | [Simulation](trading/backtesting-py/01-simulation.md) · [Data contracts](trading/backtesting-py/specs/01-data-contracts.md) |

## Study areas

| Area | Focus |
| --- | --- |
| [Backend Engineering](backend-engineering/README.md) | Studies of 12 application codebases, with a focus on system behavior and implementation. |
| [Spring](spring/README.md) | A handwritten framework-learning project and related Spring application studies. |
| [Trading](trading/README.md) | Studies of four repositories covering quantitative foundations, strategy examples, and backtesting. |

## Reading the studies

Start with a project's overview, then follow its links into focused notes. Each study keeps its original source and recorded revision where available. Coverage and completion refer to the selected study scope.

The original projects are credited to their upstream authors. This repository contains my study notes and selected source excerpts. See [Sources and studied revisions](SOURCES.md) for the complete reference list.
