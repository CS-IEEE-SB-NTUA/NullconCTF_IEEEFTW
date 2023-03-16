# Challenge

Can you stop the wheels at the right time to win?

# Solution

By loading the file in Ida we decompile the following main function:

```C
__int64 __fastcall main(unsigned int a1, char **a2, char **a3)
{
  const char *v3; // r13
  __int64 v4; // rbx
  __int64 v5; // rax
  char v6; // dl
  std::thread *v7; // rbp
  int v8; // r12d
  std::thread *v9; // rbx
  std::thread *v10; // rbp
  float *v11; // rax
  unsigned int v13; // [rsp+1Ch] [rbp-6Ch]
  int v14; // [rsp+24h] [rbp-64h] BYREF
  __int64 v15; // [rsp+28h] [rbp-60h] BYREF
  std::thread *v16[2]; // [rsp+30h] [rbp-58h] BYREF
  std::thread *v17; // [rsp+40h] [rbp-48h]

  v13 = a1;
  if ( a1 == 2 )
  {
    v3 = a2[1];
    if ( strlen(v3) == 27 )
    {
      v4 = 0LL;
      v17 = 0LL;
      v14 = 0;
      *(_OWORD *)v16 = 0LL;
      do
      {
        v7 = v16[1];
        v8 = v4;
        if ( v16[1] == v17 )
        {
          sub_1530(v16, v16[1], sub_1400, &v14, &v3[v4]);
        }
        else
        {
          *(_QWORD *)v16[1] = 0LL;
          v5 = operator new(0x18uLL);
          *(_QWORD *)v5 = &off_3D90;
          v6 = v3[v4];
          *(_DWORD *)(v5 + 12) = v4;
          *(_BYTE *)(v5 + 8) = v6;
          *(_QWORD *)(v5 + 16) = sub_1400;
          v15 = v5;
          std::thread::_M_start_thread(v7, &v15, 0LL);
          if ( v15 )
            (*(void (__fastcall **)(__int64))(*(_QWORD *)v15 + 8LL))(v15);
          v16[1] = (std::thread *)((char *)v7 + 8);
        }
        ++v4;
        v14 = v8 + 1;
      }
      while ( v4 != 27 );
      v9 = v16[0];
      v10 = (std::thread *)((char *)v16[0] + 216);
      do
      {
        std::thread::join(v9);
        v9 = (std::thread *)((char *)v9 + 8);
      }
      while ( v9 != v10 );
      v11 = (float *)&flt_40A0;
      do
      {
        if ( *v11 != 0.0 )
        {
          puts("Nope.");
          goto LABEL_16;
        }
        ++v11;
      }
      while ( v11 != (float *)((char *)&flt_40A0 + 108) );
      puts("You got it!");
      v13 = 0;
LABEL_16:
      sub_14C0(v16);
      return v13;
    }
    else
    {
      puts("Nah.");
      return 1LL;
    }
  }
  else
  {
    puts("Definitely not.");
    return 3LL;
  }
}
```

We can notice that the executable takes input from argv[1] and compares its length to 27.

If it passes the check, it creates a thread that runs inside sub_1400 with arguments v4 (which goes from 0 to 27) and v16 (which is the element of user input at index v4)

```C
void __fastcall sub_1400(int a1, char a2)
{
  float j; // xmm0_4
  int i; // eax

  if ( a2 > 0 )
  {
    j = flt_40A0[a1];
    for ( i = 0; i != a2; ++i )
    {
      for ( j = (float)(j * 5.0) + 47.0; j >= 128.0; j = j - 128.0 )
        ;
    }
    flt_40A0[a1] = j;
  }
}
```

The last function takes input from array flt_40A0 performs some operations on it and stores it again in the same position.

This is repeated for all 27 values of flt_40A0

After the thread has run the following lines of code check if each value of flt_40A0 has been set to zero

```C
v11 = (float *)&flt_40A0;
      do
      {
        if ( *v11 != 0.0 )
        {
          puts("Nope.");
          goto LABEL_16;
        }
        ++v11;
      }
 ```
 
If that is true, then we have given the correct input (the flag).

What that means is that in order to find the flag, we need to find out which input sets each float in flt_40A0 to zero.

An approach for that is to brute force each character of the input, execute function sub_1400 manually and at the end check that the respective value of flt_40A0 has become zero.

By repeating the process 27 times we can find all such characters. The final script to do this (written in c) is shown below

```C
#include <stdio.h>


int main() {
        int values[27] = {9,74,31,99,114,52,80,125,23,11,79,91,108,42,79,118,75,79,109,42,44,11,79,44,80,127,49};

        for (int elem = 0; elem < 27; elem++) {         
                for (int i = 0; i <= 150; i++) {
                        int j = values[elem];
                        for (int k = 0; k != i; k++){
                                for (j = (j * 5) + 47; j >= 128;  j = j-128);
                        }
                        if (j == 0) {
                                printf("%d ", i);
                                break;
                        }
                }
        }
}
```

By converting the resulting ints to ascii we get the flag: ENO{fl0a7s_c4n_b3_1nts_t0o}
