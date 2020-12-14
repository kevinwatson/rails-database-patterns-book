### Chapter 12 - Background Processing

## Introduction

When a large number of records need to be processed, we should use some sort of background process. Running processes in the background allows us to improve the user experience and push long-running and CPU intensive processes to times of the day or week when our compute power is idle.

There are numerous ways to provide background processing. Rails includes a framkework named `Active Job` which provides a common interface for a number of queueing backends.

## Queues

Similar to our discussion in chapter 11 about batch processing, background processing also requires strategies to prevent our application from running out of memory. One strategy is to have a process add items to a queue via Active Job and have workers that are waiting to pick up and process those jobs.

## Resources

* https://guides.rubyonrails.org/active_job_basics.html
* https://api.rubyonrails.org/classes/ActiveJob/QueueAdapters.html

## Wrap-up

