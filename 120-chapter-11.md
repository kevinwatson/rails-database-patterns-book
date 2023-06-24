### Chapter 11 - Background Processing

## Introduction

When a large number of records need to be processed, we should use some sort of background process. Running processes in the background allows us to improve the user experience and push long-running and CPU intensive processes to times of the day or week when our compute power is idle.

There are numerous ways to provide background processing. Rails includes a framkework named `Active Job` which provides a common interface for a number of queueing backends.

## Queues

Similar to our discussion in chapter 11 about batch processing, background processing also requires strategies to prevent our application from running out of memory. One strategy is to have a process add items to a queue via Active Job and have workers that are waiting to pick up and process those jobs.

### What is a Queue?

Queues are all around us. Your kitchen sink with unwashed dishes. Your garbage can with last night's takeout containers. The checkout line at the grocery store. All around us are queues that eventually flush out.

Queues come in multiple types:

* First in first out (the checkout line at the grocery store)
* First in last out (your kitchen sink)

Queues allow a system to process a large number of requests, just not all at once. Queues can be sped up by adding more workers. In our grocery store example, opening more checkout lanes can more more customers through the checkout process.

## Resources

* https://guides.rubyonrails.org/active_job_basics.html
* https://api.rubyonrails.org/classes/ActiveJob/QueueAdapters.html

[Next >>](130-chapter-12.md)
