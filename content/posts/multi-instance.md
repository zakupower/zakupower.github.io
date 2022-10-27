+++
title = "Considerations for running backend applications in multi-instance environment"
date = "2022-10-27T21:07:13+03:00"
author = "Dimitar Tomov"
description = "Some considerations to be made when running backend applications in multi-instance environment"
showFullContent = false
readingTime = false
hideComments = false

+++
Usually when you are considering running your backend application on more than one instance environment, it is because you are getting too much load for one instance to handle and want to split it between 2 or more instances, thus reducing the response times for the backend clients. You would want a load balancer between the backend instances and the backend clients, but the actual setup is beyond this post.

If you have been writing the application from the start to allow to run smoothly in such multi-instance environment that is great, but as it happened to me I needed to convert a single instance application into multiple-instance one, because of how slow response times were getting, and after analysis it turns out the database was not the problem.

## Scheduled Jobs
Scheduled jobs are to be avoided in multi-instance environment. The application I was working on relied on scheduled jobs for a lot of its operation. I would say scheduled jobs are very common in a lot of modern applications, so you have two choices.

- Move all the scheduled jobs to a second application which would act as a **worker application**. - this is what my team did, because we already had such an application, but we weren't fully utilizing it and still had jobs in our API application
- Introduce a **locking mechanism for your scheduled jobs** e.g. [Shedlock](https://github.com/lukas-krecan/ShedLock) - this is probably the choice for most application, since starting a new application that would just serve as a worker is unthinkable/unnecessary for most projects

## Database
Before you consider running an application in multiple instances you should first consider if your application is the bottle neck or the database. Most databases are very performant and would bottleneck only on very big data sets.

If your application uses **database migration scripts** e.g. [Flyway](https://flywaydb.org/), [Liquibase](https://www.liquibase.org/) etc.. these migrations could run into conflicts if multiple applications are spun at the same time and they both want to apply the same migration script to the database. Introduce some sort of way of locking when migrations are ran, so that while one instance applies migrations the other one would wait to acquire the lock, and when it acquires it will see that migrations are already applied and will skip reapplying them.

You should also look at how many **database connections your application uses**, if it is using a large number of the database's available connections (each database would have a default maximum connection property, which is usually not recommended to be increased, but can be if needed), you should consider that each instance is going to have to split those connections between them. For example, if you have 100 max database connections, and your application uses 50 of them, then the new instances shouldn't use the other 50, because you would have no connections that can be used for connecting manually to the database if you need to. Modern frameworks would reuse the connections very efficiently so just make sure your applications are not configured to hold too many connections. 

## Logging
I would say the first step would be to look at everything you are logging, and try to **reduce your logs as much as possible**.

Your logs are very important for your application, so when increasing the instance count you would have a lot more logs than with a single application. If you are not using a **log collecting, storing and searching solution** you should introduce one now. One good such solution would be the [ELK stack](https://www.elastic.co/what-is/elk-stack) which many modern applications use, which includes Elasticsearch, Logstash and Kibana for very organized way of viewing your logs (if configured correctly).

```
TODO: Continue
```