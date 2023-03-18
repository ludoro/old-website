---
layout: default
---

# System design

## Theory

### Lesson 1:
Before starting the interview:
1. Clarify the requirements of the system.
2. Back-of-the-envelope estimation for: overall scale of the system, storage, network bandwidth.
3. System interface definition: define the APIs that are expected. You'd immediately know if you got the requirements right.
4. Define the data model: NoSQL (e.g. Cassandra) vs MySQL? What kind of block storage should be used to store photos and videos?
5. High level design: Draw a block diagram with 5-6 boxes representing core components.
6. Detailed design: Dig deeper into two or three major components. 
7. Identify and resolve bottlenecks: Point of failure? Are Replicas available? How to monitor the performance of the service?



## Examples

### 1. Designing a URL Shortening service like TinyURL. 

Let's design a URL Shortening service like TinyURL. This service will provide short aliases redirecting to long URLs.

**Functional requirements**
1. Given a URL, the service should generate a shorter and unique alias of it. 
2. When users access a short link, the service should redirect them to the original link.
3. Users should optionally be able to pick up a custom short link for their URL.
4. Links will expire after a standard default timespan.

**Non-functional requirements**
1. System should be highly available, we don't want URL redirections to start failing.
2. URL redirection should be in real-time.
3. Shortened links should not be guessable. 

**Extended requirements**
1. Analytics on usage.
2. The service should be accessible through REST APIs. 

**Capacity estimation and constraints**

The system is read-heavy: lots of redirections but relatively few new URL shortenings. Assume a 100:1 ratio. 
Assume 500M new URL shortenings per month, which means 50B reads per month.
- Write QPS: 500M / (30 days * 24 hours * 3600 seconds) = 200 URLs/s.
- Read QPS: 100*200 URLs/s = 20K/s.
- Storage estimate: Assume we store a link for 5 years and the average store object is 500 bytes, then we will be storing 500M * 5 years * 12 months = 30B objects a month. This leads to 30B * 500 bytes = 15000B bytes = 15TB.
- Bandwidth estimate: For write request, we write 500 bytes: 200 * 500 bytes = 100 KB/s. For read requests, we have 20K URLs redirections, each one costing 500 bytes. Which means: 20K * 500 bytes = 10MBs.
- Memoery estimate: If we want to cache some of the hot URLs that are frequently accessed, how much memory will we need to store them? Following the 80/20 rule: 20% of the URLs generate 80% of the traffic, we'd like to cache these 20% hot URLs. We have 20K requests per second, which means 1.7 Billion request per day. Caching 20% means 170GB (0.20 * 1.7 billions * 500 bytes) in memory. There are probably many duplicate requests, so the memory usage will actually be lower. 

**System APIs**

- createURL(api_dev_key, original_url, custom_alias=None, user_name=None, expire_date=None) -> Optional(string)

- deleteURL(api_dev_key, url_key)

The API dev key is used to prevent abuse (e.g creating too many URLs).

**Database Design**

Observations: 
- We need to store billions of records.
- Each stored object is small.
- No relationship between records.
- Service is read-heavy.

Two tables needed:

<p align="center">
<b>URL</b>
</p>

|             |       |
| :---:       | :---: |
| Primary key | **Hash**   |
|  | **CreationDate**   |
|  | **OriginalURL**   |
|  | **UserID**   |


<p align="center">
<b>User</b>
</p>

|             |       |
| :---:       | :---: |
| Primary key | **UserID**   |
|  | **Name**   |
|  | **Email**   |
|  | **CreationDate**   |
|  | **LastLogin**   |

Since we need to store billions of rows and we have no relationship the clear answer is a NoSQL store like:
DynamoDB, Cassandra or Riak. It also scales horizontally much better.

**Basic System Design and Algorithm**

**First approach: Encode the actual URL**

We can encode the URL by hashing the URL and encoding. The encoding can be done in either base 32 or base 64.
A key of lenght 6 using base 64 encoding gives 64^6 possible strings, which should be more than enough. 
Using MD5 algorithm as an hash function, a 128-bit hash value is produced. After encoding in base 64, the resulting string will contain more than 21 characters. 
The reason is that each base64 character encodes 6 bit: (128bit) / (6bit/char) > 21 chars.

However, we decided on 6 characters. If we cut the string, we risk duplication. 
Moreoer, if two users enter the same URL, then they get the same shortened URL. 

We could append an increasing sequence of numbers to each input. However: overflow!

What about appending the userid? If userid is not logged in, then they need to keep retrying until they get a uniqueness key.

**Second approach: Generate keys offline**
Suppose we have a standalone Key generation service (KGS) that generates random six-letter strings beforehand and stores them in a database. 
Whenever we want to shorten a URL, we can take one already-generated key and use it. 

As soon as a key is used, It should be marked in the database to ensure that It is not used again. But, there could be race condition: if multiple servers are reading keys councurrently, then there could be a scenario where two or more servers try to read the same key from the database. 
We could use two tables to store keys: one for keys that are not used yet, the other for used keys. We can keep some keys in memory to quickly provide them whenever we need them. As soon as the keys are in memory, they are marked as used. That way, there are no race conditions. If the server dies while having those keys in memory, that is still acceptable. We also need to make sure the server synchronize (or get a lock on) the data structure holding the keys before removing keys from it and giving them to a server. 
This server is a point of failure, we can have a standby replica that can take over and provide keys.

Observation: we need to impose size limit on a custom aias to ensure a consistent URL database. 

**Data partition and replication**
We need a partitioning scheme that would divide and store our data into different DB servers. 

Solution 1: Range based partitioning. We can store URLs in separate partition based on hash key's first letter. This can lead to unbalanced partitions. 

Solution 2: Hash-based partitioning. We take an hash of the object we are storing and partition based on that. We should use consisten hashing to make sure that we don't go to unbalanced partitions. 

**Cache**
To optmize, we can cache URLs that are frequently accessed. 