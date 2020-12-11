### Chapter 11 - Batch and Background Processing

## Introduction

Batch processing is usually defined as processing large amounts of data that could be processed in a background process. An example of a background process could be something as simple as sending an email to verify a new user's email address. An example of batch processing could be sending out an email campaign to all active users on your platform.

There are numerous ways to provide background processing. Rails provides a framkework named `Active Job` which provides a common interface for a number of queueing backends.

When a large number of records need to be processed, we should use some sort of batch processing to optimize our processes.

## Resources

* https://guides.rubyonrails.org/active_job_basics.html
