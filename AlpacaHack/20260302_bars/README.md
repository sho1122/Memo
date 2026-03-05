# bars

## 問題

https://alpacahack.com/daily/challenges/bars

## 解法

指定されてる下のリンクにアクセスすると、

http://34.170.146.252:55224

FLAG: Alpaca{|1||I|l1|IIIl1|1lII|1II|1|I||||1IIl...が表示されました。

そのままコピーできなかったのでデベロッパーツールからフラグをコピーしました。

app.pyを見てみるとコピーが禁止されてるようです。
```
<body>
  <pre>FLAG: {{ flag }}</pre>
  <script>
    document.addEventListener("copy", function(e) {
      e.preventDefault();
    }, true);
  </script>
</body>

```
    


