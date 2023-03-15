# Challenge Description

# Solution

we are given the following python script:
```python
import random
import os

max_retries = 100
for _ in range(max_retries):
    print("Hints:")
    for i in range(9):
        print(random.getrandbits(32))

    real = random.getrandbits(32)
    print("Guess:")
    resp = input()
    if int(resp) == real:
        print("FLAG", os.getenv("FLAG"))

print("No tries left, sorry!")
```

The script generates 9 random 32bit numbers and asks from the user to provide the 10th. It repeats the process 100 times, generating 1000 random numbers in total.

These are more than enough for tools like **randcrack** to find the state of a non cryptographically secure random number generator like `getrandbits()` .

we write the following script:

```python
import randcrack
from pwn import *
conn = remote("52.59.124.14", 10011)

rc = randcrack.RandCrack()
count = 0
for i in range(100):
    print(conn.recvline())

    pred = 0
    for j in range(10):
        l = conn.recvline()
        if (count != 624):
            if (j == 9):
                rc.submit(0)
            else:
                rc.submit(int(l.decode().strip("\n")))
            count += 1
        if (count == 624):
            for i in range(6):
                pred = rc.predict_getrandbits(32)
            print("Pred = " + str(pred))
            print(conn.recvline())
            conn.interactive()
    conn.sendline(str(pred))

```

This script connects to the service and submits every "hint" (generated integer) in randcrack, submiting 0 for the misssing generations to keep the order theses number are generated. after 624 numbers randcrack can predict the next numbers. We generate enough numbers (6) to get the point that we need to provide the service with our prediction (630) and print hte next number. This way the number randcrack is providing is the same as the number the service expects.  We send the prediction to the service and get the flag.

Example of the runtime:
```
...
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
b'Hints:\n'  
Pred = 3603958588  
b'833825195\n'  
[*] Switching to interactive mode  
1258176177  
1292340600  
310317367  
1931763321  
Guess:  
$ 3603958588  
FLAG ENO{U_Gr4du4t3d_R4nd_4c4d3mY!}  
Hints:  
943789844  
2477013673  
4134104539  
1807591491  
1364237034  
735655207  
1209706446  
2530610248  
2134852932  
Guess:
```