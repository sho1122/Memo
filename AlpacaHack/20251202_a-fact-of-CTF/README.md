# a fact of CTF

## 問題

https://alpacahack.com/daily/challenges/a-fact-of-CTF?month=2025-12

```
# chall.py
import os

flag = os.environ.get("FLAG", "not_a_flag")

# all prime numbers less than 300
primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233, 239, 241, 251, 257, 263, 269, 271, 277, 281, 283, 293]
assert len(flag) <= len(primes)

ct = 1
for i, c in enumerate(flag):
    ct *= primes[i] ** (ord(c))

print(hex(ct))
```
## 解法

chall.pyについて

primesは素数の配列。

ord(c)は文字cのASCIIコード。例)c='A'の場合、ord(c)=65

**はべき乗。　

例えばflag="Abc"の場合、ct=primes[0]^ord('A')*3^ord('b')*5^ord('c')

hello.pyを実行するとフラグが求まる。
```
# hello.py
#print("hello")

with open("output.txt", "r", encoding="utf-8") as f:
    data = f.read()


# all prime numbers less than 300
primes = [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97, 101, 103, 107, 109, 113, 127, 131, 137, 139, 149, 151, 157, 163, 167, 173, 179, 181, 191, 193, 197, 199, 211, 223, 227, 229, 233, 239, 241, 251, 257, 263, 269, 271, 277, 281, 283, 293]

# 文字列 → 16進数 → 整数
ct = int(data, 16)

flag = ""

#素因数分解
for p in primes:
# countは指数
    count = 0
   
    while ct % p == 0:
        ct //= p
        count += 1

    if count > 0:
        flag += chr(count)

print(flag)
```
