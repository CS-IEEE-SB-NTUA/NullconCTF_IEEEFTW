# Challenge Description
WARNING: only psychics and wizards will be able complete this one

sorry :/
52.59.124.14:10012
# Solution

In this challenge we have a web service that runs a php script (index.php).
We first this line
```php
	if ($_SERVER["REQUEST_METHOD"] == "POST"
			&& $_SERVER["REQUEST_URI"] == "/submit") {
```
When we send a request that is not a post requst to the route: `/submit` (either a get request to this route or any request in another route) it execues the following:
```php
	{
		srand(random_int(0, 4294967295));

		$t = time();
		echo strval($t) . "\n";

		echo strval(rand()) . "\n";
		for ($i = 0; $i < 300; $i++) {
			if (($i % 60) == ($t % 60)) {
				echo strval(rand()) . "\n";
			} else {fit
				rand();
			}
		}

		$_SESSION["next"] = rand();
		$_SESSION["expiry"] = time() + 60;

		echo "Good luck :P";
	}
```
The previous code ( from now on Initialization) gets a (cryptographically secure) random seed from 0 to 4294967295 and generates 301 (non cryptographically secure) integers.

From those it shows on the page only the first random number and those that their index(i) is  the same modulo 60 as the time the Initialization started like so:

```
1678892907 496134670 80709468 919957814 2116517071 145427214 1291083934 Good luck :P
```

	The first number represents the timestamp (t) of the time the script started and the next numbers are those with i%60 == t%60

sets to variables for the session.  One  specifies when the sessioin expires, and is initialized to 60 seconds after the script ends and one that stores the next random number (`"next"`).

When we send a post request to the `/submit` route it executes the following:
```php
 {
		if (!isset($_SESSION["expiry"])) {
			echo "Invalid session!";
			return;
		}

		if (time() > $_SESSION["expiry"]) {
			echo "You're too slow!";
			return;
		}

		echo $_SESSION["next"]. " " . $_POST["next"] . "\n";
		if (intval($_POST["next"]) != $_SESSION["next"]) {
			echo "Wrong prediction!";
			return;
		}

		echo "FLAG " . getenv("FLAG");
```

If the session is initialized and not expired (current time < 60+ initialzation time)  it checks whether the `"next"` value of the request is the same as the `"next"` value of the session , meaning the next random number. 

This way we cannot generate enough numbers to get the state using **randcrack** like the previous challenge. We use a different method.

 We have the following script:
 

	solve.py
```python
#!/usr/bin/env python3.7
# Charles Fol
# @cfreal_
# 2020-01-04 (originally la long time ago ~ 2010)
# Breaking mt_rand() with two output values and no bruteforce.
#
"""
R = final rand value
S = merged state value
s = original state value
"""

import random
import sys

N = 624
M = 397

MAX = 0xffffffff
MOD = MAX + 1


# STATE_MULT * STATE_MULT_INV = 1 (mod MOD)
STATE_MULT = 1812433253
STATE_MULT_INV = 2520285293

MT_RAND_MT19937 = 1
MT_RAND_PHP = 0


def php_mt_initialize(seed):
    """Creates the initial state array from a seed.
    """
    state = [None] * N
    state[0] = seed & 0xffffffff;
    for i in range(1, N):
        r = state[i-1]
        state[i] = ( STATE_MULT * ( r ^ (r >> 30) ) + i ) & MAX
    return state


def undo_php_mt_initialize(s, p):
    """From an initial state value `s` at position `p`, find out seed.
    """
    # We have:
    # state[i] = (1812433253U * ( state[i-1] ^ (state[i-1] >> 30) + i )) % 100000000
    # and:
    # (2520285293 * 1812433253) % 100000000 = 1 (Modular mult. inverse)
    # => 2520285293 * (state[i] - i) = ( state[i-1] ^ (state[i-1] >> 30) ) (mod 100000000)
    for i in range(p, 0, -1):
        s = _undo_php_mt_initialize(s, i)
    return s


def _undo_php_mt_initialize(s, i):
    s = (STATE_MULT_INV * (s - i)) & MAX
    return s ^ s >> 30


def php_mt_rand(s1):
    """Converts a merged state value `s1` into a random value, then sent to the
    user.
    """
    s1 ^= (s1 >> 11)
    s1 ^= (s1 <<  7) & 0x9d2c5680
    s1 ^= (s1 << 15) & 0xefc60000
    s1 ^= (s1 >> 18)
    return s1


def undo_php_mt_rand(s1):
    """Retrieves the merged state value from the value sent to the user.
    """
    s1 ^= (s1 >> 18)
    s1 ^= (s1 << 15) & 0xefc60000
    
    s1 = undo_lshift_xor_mask(s1, 7, 0x9d2c5680)
    
    s1 ^= s1 >> 11
    s1 ^= s1 >> 22
    
    return s1

def undo_lshift_xor_mask(v, shift, mask):
    """r s.t. v = r ^ ((r << shift) & mask)
    """
    for i in range(shift, 32, shift):
        v ^= (bits(v, i - shift, shift) & bits(mask, i, shift)) << i
    return v

def bits(v, start, size):
    return lobits(v >> start, size)


def lobits(v, b):
    return v & ((1 << b) - 1)


def bit(v, b):
    return v & (1 << b)


def bv(v, b):
    return bit(v, b) >> b


def php_mt_reload(state, flavour):
    s = state
    for i in range(0, N - M):
        s[i] = _twist_php(s[i+M], s[i], s[i+1], flavour)
    for i in range(N - M, N - 1):
        s[i] = _twist_php(s[i+M-N], s[i], s[i+1], flavour)


def _twist_php(m, u, v, flavour):
    """Emulates the `twist` and `twist_php` #defines.
    """
    mask = 0x9908b0df if (u if flavour == MT_RAND_PHP else v) & 1 else 0
    return m ^ (((u & 0x80000000) | (v & 0x7FFFFFFF)) >> 1) ^ mask


def undo_php_mt_reload(S000, S227, offset, flavour):
    #define twist_php(m,u,v)  (m ^ (mixBits(u,v)>>1) ^ ((uint32_t)(-(int32_t)(loBit(u))) & 0x9908b0dfU))
    # m S000
    # u S227
    # v S228
    X = S000 ^ S227
    
    # This means the mask was applied, and as such that S227's LSB is 1
    s22X_0 = bv(X, 31)
    # remove mask if present
    if s22X_0:
        X ^= 0x9908b0df

    # Another easy guess
    s227_31 = bv(X, 30)
    # remove bit if present
    if s227_31:
        X ^= 1 << 30

    # We're missing bit 0 and bit 31 here, so we have to try every possibility
    s228_1_30 = (X << 1)
    for s228_0 in range(2):
        for s228_31 in range(2):
            if flavour == MT_RAND_MT19937 and s22X_0 != s228_0:
                continue
            s228 = s228_0 | s228_31 << 31 | s228_1_30

            # Check if the results are consistent with the known bits of s227
            s227 = _undo_php_mt_initialize(s228, 228 + offset)
            if flavour == MT_RAND_PHP and bv(s227, 0) != s22X_0:
                continue
            if bv(s227, 31) != s227_31:
                continue
            
            # Check if the guessed seed yields S000 as its first scrambled state
            rand = undo_php_mt_initialize(s228, 228 + offset)
            state = php_mt_initialize(rand)
            php_mt_reload(state, flavour)
            
            if not (S000 == state[offset]):
                continue
            
            return rand
    return None


def main(_R000, _R227, offset, flavour):
    # Both were >> 1, so the leftmost byte is unknown
    _R000 <<= 1
    _R227 <<= 1
    
    for R000_0 in range(2):
        for R227_0 in range(2):
            R000 = _R000 | R000_0
            R227 = _R227 | R227_0
            S000 = undo_php_mt_rand(R000)
            S227 = undo_php_mt_rand(R227)
            seed = undo_php_mt_reload(S000, S227, offset, flavour)
            if seed:
                return(seed)


def test_do_undo(do, undo):
    for i in range(10000):
        rand = random.randrange(1, MAX)
        done = do(rand)
        undone = undo(done)
        if not rand == undone:
            print(f"-- {i} ----")
            print(bin(rand).rjust(34))
            print(bin(undone).rjust(34))
            break

def test():
    test_do_undo(
        php_mt_initialize,
        lambda s: undo_php_mt_initialize(s[227], 227)
    )
    test_do_undo(
        php_mt_rand,
        undo_php_mt_rand
    )
    exit()


#test()

if len(sys.argv) < 5:
    print('Finds out a PHP mt_rand() seed from two outputs of mt_rand()'
          ' separated by 226 other calls.')
    print('')
    print('Usage:')
    print(f'  {sys.argv[0]} <rand_n+0> <rand_n+227> <n> <flavour>')
    print('')
    print('Parameters:')
    print('  rand_n+0:   First random value')
    print('  rand_n+227: Second random value')
    print('    The second value comes 226 mt_rand() calls after the first.')
    print('  n:          Number of mt_rand() calls in between the seeding and')
    print('              the first value (rand_n+0)')
    print('  flavour:    0 (PHP5) or 1 (PHP7+)')
else:
    main(int(sys.argv[1]), int(sys.argv[2]), int(sys.argv[3]), int(sys.argv[4]))

```
the main function of solve.py can predict the seed of the rng used by php using the first and the 227th number generated using this seed.

All we have to do now is try and get the service to generate numbers in such a timeslot that the index of the 227th (226 for the script) number has the same %60 as the timestamp, and thus is given to us.

we wrote the following script:

```python
from foresight.php import rand
import requests
from solve import main

s = requests.Session()
while(True):
	l = s.post("http://52.59.124.14:10012").text
#	print(1)
	arr = l.split('\n')
	time = int(arr[0])
	print(time % 60)
	inputs = []
	j = 1
	inputs.append(int(arr[1]))
	for i in range(300):
		if time % 60 == i % 60:
			j += 1
			if i == 226:
				inputs.append(arr[j])
				break
	if i < 250:
		print("lol")
		seed = main(int(inputs[0]),int(inputs[1]),0,7)
		print(seed)
		break
	continue

randomaki = input()

data = {"next":randomaki}

print(s.post("http://52.59.124.14:10012/submit",data=data).text)


```

this script reinitializes the state (by sending a post request to the main route instead of the / submit ) and gets the timestamp and the random generations from the response. It then runs a for loop from 0 to 300 and checks if the index that haas the same %60 as the timestamp us the desired 226. if this happened it prints the seed and sends the user input to the service.

after running this we get something like this:

```
44
44
44
44
44
45
45
45
45
45
45
45
45
45
45
45
45
45
45
45
45
46
lol
1042126681

```

In an [onine php compiler](https://onlinephp.io/) we generate the first 302 numbers using the seed we got and put the last of them in the script we wrote.
```php
 <?php
  

                  srand(1042126681);
				rand();
				  for ($i = 0; $i < 300; $i++) {
                	rand();
				  }
					echo(strval(rand()));
  
  
  ?>
  
```

After that we get the flag:

```
44
44
44
44
44
45
45
45
45
45
45
45
45
45
45
45
45
45
45
45
45
46
lol
1226534836
1283606073
FLAG ENO{M4sT3r_0f_R4nd0n0m1c5}
```
