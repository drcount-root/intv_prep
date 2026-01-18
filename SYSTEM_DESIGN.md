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

## Q2. We have a social media app like Facebook, and want to implement a feature where users can see the number of friends a person has next to each of their posts. We have a database with two tables like this: USER->[user_id(PrimaryKey), name], USER_RELATIONSHIPS->[friendship_id(Primary Key), userid_1, user_id2] where the key friendship_id is unqiue to a pair of friends. How would you implement this feature? If you could change the database implementation, what possible changes would you implement to keep this feature scalable and efficient? Keep in mind that this app will soon have more number of users than Facebook and the solution will need to scale.

Problem (Simple Version)

We have a Facebook-like app.
For every post, we want to show:

“This user has X friends”

Current Tables
USER

- user_id
- name

USER_RELATIONSHIPS

- friendship_id
- user_id_1
- user_id_2

Each row in USER_RELATIONSHIPS means two users are friends.

Naive Approach (What NOT to do)

Every time we show a post, we run:

SELECT COUNT(\*)
FROM USER_RELATIONSHIPS
WHERE user_id_1 = X OR user_id_2 = X;

Why this is bad ❌

COUNT is expensive

Feed shows many posts → many COUNT queries

With millions of users → DB gets overloaded

This will not scale

Key Idea (Very Important)

Friend count does not change often, but it is read very often.
So we should store it, not calculate it every time.

Simple & Scalable Solution ✅
Store friend_count in USER table
USER

- user_id
- name
- friend_count

Now, showing friend count is just:

SELECT friend_count FROM USER WHERE user_id = X;

✔ Very fast
✔ No counting
✔ Scales well

How is friend_count Updated?
When two users become friends

Add row to USER_RELATIONSHIPS

Increment friend count for both users

User A → friend_count +1
User B → friend_count +1

When friendship is removed

Decrement friend count for both users

Why This Works Well at Scale

Reads are O(1) (just fetch a number)

Writes happen only when friendship changes

Databases handle this pattern very well

This is how large social networks work

Optional (If interviewer asks further)

Updates can be async (event-based)

Slight delay in count is acceptable

Can cache friend_count if needed

One-Line Interview Summary ⭐

Instead of counting friends on every request, we store friend_count in the USER table and update it only when friendships change. This makes reads fast and the system scalable.
