# Some System Design QnAs

## Q1. A system is responsible for generating subtitles for videos and it is a cpu intense task. Whenever the system tries to generate subtitles for more than 10 videos at the same time, the system crashes. What is a potential cause of these crashes and how to solve this issue?

### Problem

Subtitle generation is CPU-heavy

The system tries to process too many videos at the same time

There is no limit on parallel processing

CPU gets overloaded

When CPU is fully exhausted → system crashes

👉 The main issue is uncontrolled concurrency on a CPU-bound task

### Root Cause Analysis

Each subtitle job consumes a lot of CPU

Running many jobs in parallel:

Causes CPU contention

Excessive context switching

Threads starve

The system tries to “do everything at once” instead of pacing the work

### Solution

1. **Use an asynchronous job queue**

   Do not generate subtitles immediately

   Push subtitle requests into a queue

   Examples: Kafka / SQS / RabbitMQ

   This decouples request intake from processing

2. **Limit concurrent processing with workers**

   Have a fixed number of workers

   Each worker processes one video at a time

   Example:

   Only process 5 videos concurrently

   Remaining requests wait in the queue

3. **Scale safely if needed**

   Add more workers or machines when load increases

   Queue distributes work automatically

   No sudden CPU spikes

### Why this works?

Queue smooths traffic spikes

Worker limit protects CPU

Extra requests wait instead of crashing

System becomes stable and scalable

### Summary

The crash occurs because the system runs too many CPU-intensive subtitle jobs in parallel. Using an asynchronous queue with a limited worker pool controls concurrency, prevents CPU overload, and keeps the system stable.

---

## Q2. 