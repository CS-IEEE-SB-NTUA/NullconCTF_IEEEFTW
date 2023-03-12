# Challenge Description

Can you find your way through Pythopia? But you need a valid license to enter the city first.

# Solution

We are presented with a python ast file consisting of some flag (license) checkers. Let's break them down:

### 1st Checker

The first checker checks our flag against some constant input:

![checker1](/images/checker1.png)

Gathering all these values we see that the first part of the flag is ENO{L13333333333

### 2nd Checker

The second checker checks our flag against some constant values, after XORing them with number 19
