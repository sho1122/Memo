# Emojify

## 問題

https://alpacahack.com/daily/challenges/emojify?month=2025-12

```
# frontend/index.js
import express from "express";
import fs from "node:fs";

const waf = (path) => {
  if (typeof path !== "string") throw new Error("Invalid types");
  if (!path.startsWith("/")) throw new Error("Invalid 1");
  if (!path.includes("emoji")) throw new Error("Invalid 2");
  return path;
};

express()
  .get("/", (req, res) => res.type("html").send(fs.readFileSync("index.html")))
  .get("/api", async (req, res) => {
    try {
      const path = waf(req.query.path);
      const url = new URL(path, "http://backend:3000");
      const emoji = await fetch(url).then((r) => r.text());
      res.send(emoji);
    } catch (err) {
      res.send(err.message);
    }
  })
  .listen(3000);

```

```
# backend/index.js
import express from "express";
import * as emoji from "node-emoji";

express()
  .get("/emoji/:text", (req, res) =>
    res.send(emoji.get(req.params.text) ?? "❓")
  )
  .listen(3000);
```
```
# secret/index.js
import express from "express";

const FLAG = process.env.FLAG ?? "Alpaca{REDACTED}";

express()
  // http://secret:1337/flag
  .get("/flag", (req, res) => res.send(FLAG))
  .listen(1337);

```

## 解法

pizzaと入力してボタンを押すと

<img width="653" height="505" alt="image" src="https://github.com/user-attachments/assets/07251e9e-23c3-455a-af21-c3d1701ead33" />

pizza以外の文字を入力してボタンを押すと

<img width="664" height="503" alt="image" src="https://github.com/user-attachments/assets/0576d97b-2034-4cd8-97bb-0ba83c98c7ee" />

api?path= は、HTTPリクエストクエリパラメータ。

以下のようにアクセスすると、

<img width="416" height="73" alt="image" src="https://github.com/user-attachments/assets/bb07c7e9-bd96-42b4-a8aa-25c31a56937f" />

pathにemojiがないとはじかれるので、以下のようにアクセスすると、

<img width="460" height="75" alt="image" src="https://github.com/user-attachments/assets/b8c030aa-d297-4842-b240-ba2f3a83a095" />


