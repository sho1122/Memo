# 100

## 問題

https://alpacahack.com/daily/challenges/100

## 解法

100.pyについて
```
import os

# 空の配列
I00 = set()
# 100回ループ
for L00 in range(100):
    # 100?と表示して入力を受け取る。受け取った文字は.strip()で空白削除
    l00 = input("100? ").strip()
    # 入力文字がASCII文字か
    assert l00.isascii()
    # 入力文字が10文字以内か
    assert len(l00) <= 10.0
    # 今まで入力した文字は×
    assert l00 not in I00
    # 入力した文字を整数に変換すると100であるか
    assert int(l00) == 100
    # 配列に入力文字追加
    I00.add(l00)

print(os.getenv("FLAG", "Alpaca{REDACTED}"))
```

仮想環境を作成(hello.pyでfrom pwn import *を使うため)
```
python3 -m venv venv
source venv/bin/activate
pip3 install pwntools
```

hello.pyを実行するとフラグを得られる
```
import itertools

chars = ['+', '0', '1', '_']
I00 = []

# +,0,1,_を使って100を表現する。それを配列I00に入れる。 例)+1_00
for l in range(3, 10):
    # itertools.product()は全組み合わせを作る。l=3の場合+++,++0,++1....
    for p in itertools.product(chars, repeat=l):
        # p = ('1','0','0')をs="100"にする
        s = "".join(p)
        try:
            if int(s) == 100:
                if s not in I00: I00.append(s)
        except:
            pass

print(len(I00)) #配列数
print(I00)

import os
import secrets
import signal
from pwn import *

HOST = "34.170.146.252"
PORT = 23793

#サーバ接続
r = remote(HOST,PORT)
i=0
while True:
    print("i =", i)

    try:
        r.recvuntil(b"100?")
        print("100?")
        #r.sendline(100)

        #100を送る
        r.sendline(I00[i].encode())
    except EOFError:
        print("server closed connection")
        break

    if(i==99):
        #フラグ出力
        print(r.recvline().decode()) 
        break

    i=i+1
```
