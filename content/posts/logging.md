+++
title = "Turns out Logging is hard"
date = "2022-11-23T22:13:30+02:00"
author = "Dimitar Tomov"
tags = ["logging", "experience"]
keywords = ["java", "logging"]
description = "Why is logging so hard?"
showFullContent = false
readingTime = false
hideComments = false

+++

Turns out logging is hard! When I was a green junior developer I thought nothing of logging. It was just something that I knew I had to do. Rule I was following was, if exception occurs log it and rethrow if necessary. As it turns out that is the dumbest rule ever. Filling your logs with exceptions which are 1. already logged by application and 2. there is nothing useful about seeing them in the logs is unnecessary.

## How it began

I stumbled upon this issue with an application that was being used by 300+ users at the same time, and these users were very active in specific timeslots 8:30 am to 12:00 pm and 16:00 pm to 20:00 pm. When application was released on PROD there were so many other issues that were being reported on the daily, and all were critical so we didn't even notice how our logs went from 10 archived files per day to **100 to 900 archived files for 1 day**.

 ## What were the issues

After our logs took a turn for the worse, we started doing analysis on what the issues were. 

### Request logging

We wanted to keep track of the calls to our endpoints, and have them all stored for future analysis. Also we had some very tricky bugs which we wanted to pin point this way. We had our request logs turned on from day 1 of PROD Release. Turns out from all these request logging our application was being slowed down a lot. After discussing with POs and the client, we found out that we assumed (wrongly) we wanted to keep track and store all endpoints, but it turned out we only needed one endpoint. So we turned off all logs for other endpoints and kept this one.

After some time, these logs were annoying to scroll through when looking at the other logs of the application, so we even moved this endpoint's logs to a different log file structure with its own RollingAppender. This significantly improved the situation, and we were happy with our decisions. Now we had a separate log file structure, which we could analyze if needed for this endpoint.

### The obsolete error logging

This was not enough for us, we still had 100s of archived log files for a day. So we went on to the next culprit obsolete error logs

We gathered the logs from a single day of the application, and collected all errors which were logged. And we asked ourselves the question:

- Is this programmer or client error?
  - If programmer error -> Fix it
  - If client error -> Truncate the error log as much as possible

There were multiple instances of validation errors, that we were just logging the whole stack trace and never using it for anything. These error were truncated to simple one line logs.

### Unexpected "DDOS"

This issue is something we should have noticed first, but we were ignoring the obvious. The most frequent error in our logs was a Unique Constraint Violation, that was happening on the main (most used) table of the application. 

Our most used endpoint, the one we were logging the requests for was an async operation, that validated the input without any DB hits and placed the request in a queue to be processed and return 200 OK, so the user doesn't have to wait on a very expensive chain of operations that would happen after this endpoint is called. The Frontend application had a retry mechanism (because the information for this endpoint was very important to never be lost) that would call this endpoint. For some reason the retry mechanism was not working properly and was retrying the same resource 1000s of times per day. So except the traffic we were getting from users, we were also getting these retry requests based on local storage in the frontend. When we did the analysis it turned out in the span of 1 week, we were being sent 1000s of resources 1000s of times a day which were all failing with Unique Constraint Violation. 

We were being "DDOS-ed" by our own Frontend application, and this filled the logs to the brim and also slowed down the Backend application significantly (an issue we were also investigating in parallel to the logs).

The solution for this was fixing the Frontend Retry mechanism, and also adding a simple (and fast) DB call which would check for already existing resource on the specific endpoint and before queueing the resource for processing throwing a 422 UNPROCESSABLE ENTITY error to the frontend so it can delete it from the local storage.

## Lessons learned

TODO("Write this!")

## Useful Articles

[To Log, or Not to Log - An Alternative Strategy to Make Loggers your Friends (by Stanley Nguyen)](https://www.freecodecamp.org/news/how-to-use-logs-effectively-in-your-code)

[The Problem With Logging (by Jeff Atwood)](https://blog.codinghorror.com/the-problem-with-logging/)

[Do not log (by Nikita Sobolev)](https://sobolevn.me/2020/03/do-not-log)