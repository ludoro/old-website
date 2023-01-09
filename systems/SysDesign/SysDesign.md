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
