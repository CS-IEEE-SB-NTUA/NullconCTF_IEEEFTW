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

![checker2_1](/images/checker2_1.png)
![checker2_2](/images/checker2_2.png)

So after performing correct XORs we get the second part: 7_super_duper_ok

### 3rd Checker

The third checker reverses our flag and compares it with the string "_!ftcnocllunlol_"

![checker3](/images/checker3.png)

So the third part is _lolnullconctf!_

### 4th Checker

Last checker just compares the final part of the flag against a constant string: "you_solved_it!!}"

![checker4](/images/checker4.png)

### Putting it all together

Putting it all together we get the flag: ENO{L133333333337_super_duper_ok_lolnullconctf!_you_solved_it!!}

