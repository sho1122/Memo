# Find XOR key

## 問題

https://alpacahack.com/daily/challenges/find-xor-key

```
import os
import secrets
import string
from itertools import cycle

flag = os.getenv("FLAG", "Alpaca{FAKEFAKEFAKEFAKE}").encode()
assert flag.startswith(b"Alpaca{")

# key = b"???????", e,g, abcdefg
key = b"".join(secrets.choice(string.ascii_letters).encode() for _ in range(7))
assert len(key) == 7

c = bytes([c1 ^ c2 for c1, c2 in zip(flag, cycle(key))])
print(c.hex())
```

c = bytes([c1 ^ c2 for c1, c2 in zip(flag, cycle(key))])の意味
for c1, c2 in zip(flag, cycle(key)):
    c1 ^ c2
    
c1 = flag[i]
c2 = key[i % len(key)]

cの1文字目 = flag[0]^key[0 % len(key)]
cの2文字目 = flag[1]^key[1 % len(key)]
....

hello.pyを実行するとFLAGがもとまる
```
import string
from itertools import cycle

with open("output.txt", "r", encoding="utf-8") as f:
    c_string = f.read()

c = bytes.fromhex(c_string)
alpaca = b"Alpaca{"

#keyは7文字
key = bytearray(7)

for i in range(7):
    c1 = c[i]
    c2 = alpaca[i]
    key[i] = c1^c2

print(key)
# key = b'BwcfNIq'

flag = bytes([c1 ^ c2 for c1, c2 in zip(c, cycle(key))])
print(flag)
```
