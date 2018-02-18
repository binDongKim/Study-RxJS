# Core Concepts in RxJS - Part 2 - Scheduler

## I will cover Scheduler later when I fully understand it.

## What is Scheduler?

- A Scheduler controls when a subscription starts and when notifications are delivered.
- It consists of three components:
  - **Data Structure**: It knows how to store and queue tasks based on priority or other criteria.
  - **Execution Context**: It denotes where and when the task is executed(e.g. immediately, or in another callback mechanism).
  - **(Virtual) Clock**: It provides a notion of "time" by a getter method `now()` on the scheduler. Tasks being scheduled on a particular scheduler will adhere only to the time denoted by that clock.
- A Scheduler lets you define in what execution context will an Observable deliver notifications to its Observer.

## References

- [Scheduler in RxJS](http://reactivex.io/rxjs/manual/overview.html#scheduler)

