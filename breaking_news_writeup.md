# Challenge Description

Alice started to encrypt the flag, but realised halfway she was unhappy with her key. So she created a new one.
# Solution
We are given the flag split in two ciphertexts $$c_1, c_2$$ each encrypted with a different key. BUT, both keys use the same public modulus $$n$$(and different $$e, d$$).

We notice the public exponents e1, e2 are very large. This can sometimes mean the respective private exponents are small enough to be vulnerable to Wiener's attack or Boneh-Durfee attack. We tried a sage script we found online for Boneh-Durfee and indeed the first key was vulnerable giving us the private exponent   
$$d_1 = 3142948387612230061712223313218058768177264054157930977501720905159512174225$$   

The second private exponent couldn't be found the same way but we can use the pair $$d_1, e_1$$ to [factor](https://www.di-mgt.com.au/rsa_factorize_n.html)  $$n$$.
After factoring $$n$$ we can easily use the other public exponent $$e_2$$ to calculate $$d_2 = e^{-1}_2 mod (p-1)(q-1)$$ where $$p,q$$ are the factors of $$n$$.

We load the keys we found with the correct 'mode'(PKCS1_OAEP) and simply decrypt the two parts of the flag to get: ENO{n3ver_reus3_your_pr1mes_4_a_new_k3y_you_have_2_p4y_th3_pr1ce}









