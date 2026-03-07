# high and low

## 問題

https://alpacahack.com/daily/challenges/high-and-low?month=2026-03

```
import os
import secrets
import signal

class RNG:
    N = 624
    M = 397
    UPPER_MASK = 0x80000000
    LOWER_MASK = 0x7FFFFFFF

    def __init__(self):
        self.state = [secrets.randbits(32) for _ in range(self.N)]
        self.p = 0

    def next_value(self):
        p, q, r = self.p, (self.p+1) % self.N, (self.p + self.M) % self.N
        a = self.state[p] & self.UPPER_MASK
        b = self.state[q] & self.LOWER_MASK
        x = (a | b) ^ self.state[r]

        self.state[p] = x
        self.p = q

        y = ((x >> 11) | ((x << 21) & 0xFFFFF800)) ^ 0xDEADBEEF
        return y

signal.alarm(600)

rng = RNG()

money = rng.N
while True:
    print(f"money: {money}")
    if money < 0:
        print("bankrupt!")
        exit()
    if money > 1337:
        flag = os.environ.get("FLAG", "Alpaca{REDACTED}")
        print("rich man!")
        print(flag)
        exit()

    value = rng.next_value()
    print(f"value: {value}")

    choice = input("high or low? ")
    print(f"[{choice}]")

    next_value = rng.next_value()
    print(f"next: {next_value}")
    if  (choice == "h") == (value < next_value):
        print("you win")
        money += 1
    else:
        print("you lose")
        money -= 1
```

## 解法

0x80000000を2進数で表すと
10000000 00000000 00000000 00000000

0x7FFFFFFFを2進数で表すと
01111111 11111111 11111111 11111111

secrets.randbits(32) : 32bitのランダム整数

self.state = [secrets.randbits(32) for _ in range(self.N)]は、

state[0], state[1], ..., state[N-1] が全部ランダムな 32bit 値になる。

```
p, q, r = self.p, (self.p+1) % self.N, (self.p + self.M) % self.N
```
p : 現在の index

q : 次の index

r : M だけ先の index

```
a = self.state[p] & self.UPPER_MASK
```
a : state[p] の上位 1 ビットを取り出す

```
 b = self.state[q] & self.LOWER_MASK
```
b : state[q] の下位 31 ビットを取り出す
```
x = (a | b) ^ self.state[r]
```
aとbのORをstate[r]でXOR。twist（ねじり）処理。

```
self.state[p] = x
self.p = q
```
state[p] をxの値に更新。pをqの値に更新。
```
y = ((x >> 11) | ((x << 21) & 0xFFFFF800)) ^ 0xDEADBEEF
```
x >> 11 : 右に 11 ビットシフト

((x << 21) & 0xFFFFF800))  : 左に 21 ビットシフトして、上位ビットだけ残す


from pwn import *を使う際のためにBashで以下を実行
```
sudo apt install python3.12-venv
python3 -m venv venv
source venv/bin/activate
pip3 install pwntools
```
※仮想環境から抜ける場合
```
deactivate
```

以下を実行してフラグを求める。
```
import os
import secrets
import signal
from pwn import *

HOST = "34.170.146.252"
PORT = 6881

class RNG:
    N = 624
    M = 397
    UPPER_MASK = 0x80000000
    LOWER_MASK = 0x7FFFFFFF

    # def __init__(self):
    #     self.state = [secrets.randbits(32) for _ in range(self.N)]
    #     self.p = 0

    def __init__(self,state):
        self.state = state[:]
        self.p = 0

    def next_value(self):
        p, q, r = self.p, (self.p+1) % self.N, (self.p + self.M) % self.N
        a = self.state[p] & self.UPPER_MASK
        b = self.state[q] & self.LOWER_MASK
        x = (a | b) ^ self.state[r]

        self.state[p] = x
        self.p = q

        y = ((x >> 11) | ((x << 21) & 0xFFFFF800)) ^ 0xDEADBEEF
        return y

signal.alarm(600)

# rng = RNG()

# money = rng.N
# while True:
#     print(f"money: {money}")
#     if money < 0:
#         print("bankrupt!")
#         exit()
#     if money > 1337:
#         flag = os.environ.get("FLAG", "Alpaca{REDACTED}")
#         print("rich man!")
#         print(flag)
#         exit()

#     value = rng.next_value()
#     print(f"value: {value}")

#     choice = input("high or low? ")
#     print(f"[{choice}]")

#     next_value = rng.next_value()
#     print(f"next: {next_value}")
#     if  (choice == "h") == (value < next_value):
#         print("you win")
#         money += 1
#     else:
#         print("you lose")
#         money -= 1

# ------------------------
# collect outputs
# ------------------------
r = remote(HOST,PORT)
outputs = []
N = 624
i=0
icount=0
while True:
    print("i =", i)

    try:
        r.recvuntil(b"money: ", timeout=1)
        money = int(r.recvline().strip())
        print(f"money: {money}") 
    except EOFError:
        print("server closed connection")

    try:
        r.recvuntil(b"value: ")
        value = int(r.recvline())
        print(f"value: {value}") 
        outputs.append(value)
        icount = icount + 1
        print("icount =", icount)
        if(icount==N): break
        
    except EOFError:
        print("server closed connection")

    try:
        r.recvuntil(b"high or low?")
        r.sendline(b"h")
    except EOFError:
        print("server closed connection")

    try:
        r.recvuntil(b"next: ")
        next_value = int(r.recvline())
        print(f"next_value: {next_value}") 
        outputs.append(next_value)
        icount = icount + 1
        print("icount(next) =", icount)
        if(icount==N): break
        
    except EOFError:
        print("server closed connection")

    i=i+1   

# ------------------------
# recover
# ------------------------       
def recover(y):
    z = y ^ 0xDEADBEEF
    x = ((z << 11) | (z >> 21)) & 0xFFFFFFFF
    return x

# ------------------------
# recover state
# ------------------------
state = [recover(o) for o in outputs]
rng = RNG(state)

# ------------------------
# send h or l
# ------------------------
p = 0
while True:
    print("p =", p)
    value = rng.next_value()
    next_value = rng.next_value()

    # try:
    #     r.recvuntil(b"money: ", timeout=1)
    #     money = int(r.recvline())
    #     print(f"money: {money}") 
    # except EOFError:
    #     print("server closed connection")

    # try:
    #     r.recvuntil(b"value: ")
    #     r.recvline()
    # except EOFError:
    #     print("server closed connection")

    try:
        r.recvuntil(b"high or low?")
        if value < next_value:
            r.sendline(b"h")
        else:
            r.sendline(b"l")  
    except EOFError:
        print("server closed connection")

    # try:
    #     r.recvuntil(b"next: ")
    #     r.recvline()
    # except EOFError:
    #     print("server closed connection")

    try:
        line = r.recvline()

        if not line:
            break

        if b"rich man!" in line:
            flag = r.recvline()
            print(flag.decode().strip())
            break
        else:
            # print(r.recvline().decode())  
    except EOFError:
        print("server closed connection")

    # try:
    #     print(r.recvline().decode())  
    # except EOFError:
    #     print("server closed connection")

    p=p+1

```



