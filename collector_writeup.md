# Challenge Description

Bob is hosting a party and invited everyone but me. But all the invitations I can collect are encrypted.

# Solution
The challenge files contain the source code.
We can see this is another RSA challenge that includes some weird custom padding:
```python
def MGF(seed, length):
    random.seed(seed)
    return [random.randint(0,255) for _ in range(length)]

def pad(stream, bitlength, tag = b''):
    seed = sha256(stream).digest()
    DB = sha256(tag).digest() + b'\x00' * ((bitlength - 16 - 2*hLen) // 8 - len(stream)) + b'\x01' + stream
    mask = MGF(seed, len(DB))
    maskedDB = [DB[i] ^ mask[i] for i in range(len(DB))]
    seedmask = MGF(bytes_to_long(bytes(maskedDB)), hLen // 8)
    masked_seed = [seed[i] ^ seedmask[i] for i in range(hLen // 8)]
    EM = [0] + masked_seed + maskedDB
    return bytes(EM)

key = RSA.generate(N, e = (1<<(1<<random.randint(0,4))) + 1)
msg = pad(flag, N)
cipher = pow(bytes_to_long(msg), key.e, key.n)
print(key.publickey().export_key())
print(hex(cipher))
```
Another key(hehe pun intended) observation is that the RSA key is generated semi-randomly each time. Specifically the public modulus N is a random 2048-bit composite but the public exponent e takes a random value from the set: {3, 5, 17, 257, 65537}.
Since the same message is encrypted each time and we can get as many encryptions as we want from the server, all we have to do is get 3 encryptions that used $$e=3$$ as their public exponent(or similarly 5 that uses $$e=5$$,  17 that used $$e=17$$ etc ...).
Then we have three congruences:
$$m^3=c_1 \: mod \: n_1$$
$$m^3=c_2 \: mod \: n_2$$
$$m^3=c_3 \: mod \: n_3$$
Using the chinese remainder theorem we can calculate $$m^3$$ from these three congruences and 
since for RSA all messages are smaller than the modulus ($$ m < n_i $$) it follows that $$m^3 < n_1\ast n_2\ast n_3$$ so we are guaranteed to have the correct answer.

Now given $$m^3$$ we can compute $$m$$ simply by calculating the cube root over the integers.
But this m is not the flag! Don't forget we have to undo all that padding. Thankfully, it's completely reversible.
We split the padded message m into [0] + masked_seed + maskedDB where masked_seed is 32 bytes.
We use the given MGF() function to calculate
```python
seedmask = MGF(bytes_to_long(bytes(maskedDB)), hLen // 8)
```
and with seedmask + masked_seed we can calculate the seed:
```python 
seed = [masked_seed[i] ^ seedmask[i] for i in range(hLen // 8)]
 ```
Finally, using seed and maskedDB we can can get the original DB(which contains the flag):
```python
seed = sha256(stream).digest()
mask = MGF(seed, len(DB))
DB = [maskedDB[i] ^ mask[i] for i in range(223]
```
If we print DB we see this array:
```python
[227, 176, 196, 66, 152, 252, 28, 20, 154, 251, 244, 200, 153, 111, 185, 36, 39, 174, 65, 228, 100, 155, 147, 76, 164, 149, 153, 27, 120, 82, 184, 85, 0, 0, 0, 0, 
0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 69, 78, 79, 123, 99, 111, 109, 51, 95, 116, 111, 95, 78, 117, 49, 108, 67, 111, 110, 95, 98, 117, 49, 95, 100, 111, 95, 110, 48, 116, 95, 116, 101, 108, 49, 95, 66, 48, 98, 125]
```
And by converting the last part into ascii characters we finally get the flag:
ENO{com3_to_Nu1lCon_bu1_do_n0t_tel1_B0b}

