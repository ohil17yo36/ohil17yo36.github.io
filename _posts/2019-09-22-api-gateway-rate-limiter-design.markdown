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
4. It takes in the `rateLimiter` as first argument, ` total number of requests` as second argument, the `request rate`(requests per second) as the third argument

Now let's look at each `sendRequest()` test