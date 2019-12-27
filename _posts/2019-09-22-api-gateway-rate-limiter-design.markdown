---
layout: post
title:  "Api Gateway Rate Limiter Design"
date:   2019-09-22 13:26:47 -0700
categories: api gateway rate limiter
---

### High level requirement
Imagine you are tasked to design an API gateway to `rate limit` your exposed public API's.


### First thoughts
1. Consider this as an algorithm question, If you even have a ratelimiter then how are we going to test it?