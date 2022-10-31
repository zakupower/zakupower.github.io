+++
title = "Considerations for running backend applications in multi-instance environment"
date = "2022-10-27T21:07:13+03:00"
author = "Dimitar Tomov"
description = "Some considerations to be made when running backend applications in multi-instance environment"
showFullContent = false
readingTime = false
hideComments = false

+++
## Scheduled Jobs
Scheduled jobs are to be avoided in multi-instance environment. The application I was working on relied on scheduled jobs for a lot of its operation. I would say scheduled jobs are very common in a lot of modern applications, so you have two choices.

- Move all the scheduled jobs to a second application which would act as a **worker application**. - this is what my team did, because we already had such an application, but we weren't fully utilizing it and still had jobs in our API application
- Introduce a **locking mechanism for your scheduled jobs** e.g. [Shedlock](https://github.com/lukas-krecan/ShedLock) - this is probably the choice for most application, since starting a new application that would just serve as a worker is unthinkable/unnecessary for most projects

## Logging
I would say the first step would be to look at everything you are logging, and try to **reduce your logs as much as possible**.

Your logs are very important for your application, so when increasing the instance count you would have a lot more logs than with a single application. If you are not using a **log collecting, storing and searching solution** you should introduce one now. One good such solution would be the [ELK stack](https://www.elastic.co/what-is/elk-stack) which many modern applications use, which includes Elasticsearch, Logstash and Kibana for very organized way of viewing your logs (if configured correctly).

```
TODO: Continue
```