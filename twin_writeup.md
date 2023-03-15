# Challenge Description

The twins Alice and Bob are so close that they share everything, even the modulus of their RSA keys.

# Solution
We are given two RSA public keys (e1, n), (e2, n) and two cipheretexts c1, c2 of the flag encrypted with each of the two keys. 
$$c_1=m^{e_1}\:mod\:n$$
$$c_2=m^{e_2}\:mod\:n$$

Next we calculate the gcd of the the two exponents and we find gcd(e1, e2) = 17.
$$e_1'=\frac{e_1}{17}$$
$$e_2'=\frac{e_2}{17}$$
From Bezout's identity we can find the proper u, v such that:
$$u*e_1'+v*e_2'=1$$
Now we can see the following holds:
$$c_1^{u}*c_2^{v}=m^{e_1*u}*m^{e_2*v}=m^{17*(u*e_1'+v*e_2')}={(m^{u*e_1'+v*e_2'})}^{17}=(m^1)^{17}=m^{17}$$
After performing the above caclulations using the two ciphertexts and u, v we try to get the 17th root of the new ciphertext. Thankfully, 17 is a small enough exponent and we get the correct m.
Finally we convert to bytes and get the flag: b'ENO{5har1ng_is_n0t_c4r1ng}'!

