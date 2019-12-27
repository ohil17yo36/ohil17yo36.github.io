---
layout: post
title:  "Api Gateway Rate Limiter Design"
date:   2019-09-22 13:26:47 -0700
categories: api gateway rate limiter
---

## High level requirement
Imagine you are tasked to design an API gateway to `rate limit` your exposed public API's.


## First thoughts
1. Consider this as an algorithm question, How would you even design a `rate limiter`?
2. What would be the input and output to such an algorithm?
3. If you even have a rate limiter then how are we going to test it's performance on load?
4. What are the different policies used in rate limiters?

## Learnings
[Credits](https://hechao.li/2018/06/25/Rate-Limiter-Part1/)

### Interface
Let's first design the basic interface of a rate limiter

```java
public abstract class RateLimiter {

  protected final int maxRequestPerSec;

  protected RateLimiter(int maxRequestPerSec) {
    this.maxRequestPerSec = maxRequestPerSec;
  }

  abstract boolean allow();
}
```
1. We have an abstract class called RateLimiter which taken in the maximum allowed hit rate as it's constructor parameter.
2. We have a method interface called `allow()` that returns a boolean which tells the caller whether this request is allowed or dropped by the gateway

### Testing
```java
public class Solution {
    
    public static void sendRequest(RateLimiter rl, int totalRequests, int requestRate) {
     
        long startTime = System.currentTimeMillis();
        CountDownLatch cdl = new CountDownLatch(totalRequests);
 
        for(int i=0; i<totalRequests; i++) {
            new Thread(() -> {
                while(!rl.allow()) {
                    try{
                        Thread.sleep(10);
                    } catch(InterruptedException ie) {
                        throw new RuntimeException(ie);
                    }
                }
                cdl.countDown();
            }).start();
            
            try{
                Thread.sleep(1000/requestRate);
            } catch(InterruptedException ie) {
                throw new RuntimeException(ie);
            }
        }
 
        try{
            cdl.await();
        } catch(InterruptedException ie) {
            throw new RuntimeException(ie);
        }
 
 
        double duration = (System.currentTimeMillis() - startTime) / 1000.0;
        System.out.println(totalRequests + " requests processed in " + duration + " seconds. " + "Rate: " + (double) totalRequests / duration + " per second");
     
    }
    
    public static void main (String[] args) throws java.lang.Exception
    {
        // your code goes here
        RateLimiter rl = new ImplementationBasedRateLimiter(20);
        Thread thread = new Thread(() -> {
                sendRequest(rl, 10, 1);
                sendRequest(rl, 20, 2);
                sendRequest(rl, 50, 5);
                sendRequest(rl, 100, 10);
                sendRequest(rl, 200, 20);
                sendRequest(rl, 250, 25);
                sendRequest(rl, 500, 50);
                sendRequest(rl, 1000, 100);
            });
 
        thread.start();
        thread.join();
    }
}
```
Ok, there is a lot of things going on here. Let me disect them and explain one by one.

Let's look at the main function
1. First the main function initializes a rate limiter
2. Second, the main function prepares a batch of tests. 
3. Each `sendRequest()` method is a test. 
4. It takes in the `rateLimiter` as first argument,`total number of requests` as second argument, the `request rate`(requests per second) as the third argument

Now let's look at each `sendRequest()` test
1. The objective of this method is to launch `totalRequests` number of requests against the rate limiter at the `request rate` pace.
2. To do that we can send each request after a time interval of `(1000/requestRate)` milliseconds. Simple math :\)
3. This is achieved by iterating over a for loop and waiting for (1000/requestRate) milliseconds after every iteration
4. In each iteration, if we simply launch a new thread that checks for the rate limiters `allow()` method to be true just once, then we are not replicating real testing scenario.
5. In each iteration, the thread launched has to keep trying the allow() method of the rate limiter untill it returns true.
6. Hence we use a `while loop` in the launched thread of each iteration
7. Now, is that enough. Nope, because the main thread needs to wait till all the threads have finished execution.
8. Best way to ensure that is to use a `CountDownLatch` and let the main thread wait till all the threads have decremented the latch to zero.
9. The remaining part is simply measuring `execution metrics` and printing to the console.