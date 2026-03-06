# Flag Printer 2026

## 問題
https://alpacahack.com/daily/challenges/flag-printer-2026

```
# server.py
import time

flag = "Alpaca{s???}"
assert len(flag) == 12

for i, c in enumerate(flag):
    print(c, end="", flush=True)
    time.sleep(i)
```

```
# Dockerfile
FROM python:3.14.3
WORKDIR /app
RUN apt-get update && apt-get install -yq socat
COPY server.py .

CMD ["socat", "-T5", "tcp-listen:1337,fork,reuseaddr", "exec:'python server.py'"]
```

```
# compose.yaml
services:
  challenge:
    build: .
    restart: unless-stopped
    ports:
      - ${PORT:-1337}:1337
```

## 解法

以下のように接続すると、"Alpaca{"までは表示されますが、その後ろは表示されません。
```
nc 34.170.146.252 10951
```

Dockerfileを見ると以下のようになってます。
```
# Dockerfile
CMD ["socat", "-T5", "tcp-listen:1337,fork,reuseaddr", "exec:'python server.py'"]
```

socat : いろいろな通信を橋渡しするツール

-T5 : 5秒間データの送受信が無ければ接続を終了。

tcp-listen:1337 : TCP 1337ポートで待ち受け。nc localhost 1337というように接続できる。

fork : 接続ごとに新しいプロセスを作る。

reuseaddr : ポートをすぐ再利用できる設定。

exec:'python server.py' : 接続したらpython server.pyを実行する。


Dockerfileを見ると5秒接続がないと切断されてしまうので、1秒ごとに「.」をサーバーに送り続けて接続を切られないようにします。下のように接続するとフラグが返ってきます。
```
while true; do echo "."; sleep 1; done | nc 34.170.146.252 10951
```

