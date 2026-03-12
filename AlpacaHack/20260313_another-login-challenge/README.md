# Another Login Challenge

## 問題

https://alpacahack.com/daily/challenges/another-login-challenge

## 解法

index.js
```
import express, { urlencoded } from "express";
import crypto from "crypto";

const FLAG = process.env.FLAG ?? "Alpaca{REDACTED}";

const app = express();
app.use(urlencoded({ extended: false }));

#パスワードはランダムの数字
let users = {
  admin: {
    password: crypto.randomBytes(32).toString("base64"),
  },
};

#ログインフォーム
app.get("/", (req, res) => {
  res.send(`
    <h2>Login</h2>
    <form method="POST">
      <input name="username" placeholder="username" required />
      <br />
      <input name="password" type="password" placeholder="password" required />
      <br />
      <button>Login</button>
    </form>
  `);
});

#パスワードが一致してたらフラグが表示される
app.post("/", (req, res) => {
  const { username, password } = req.body;
  const user = users[username];
  if (!user || user.password !== password) {
    return res.send("invalid credentials");
  }

  res.send(FLAG);
});

app.listen(3000, () => {
  console.log("http://localhost:3000");
});

```

http://localhost:3000にアクセスしてみるとログインフォームが表示される。

<img width="275" height="158" alt="image" src="https://github.com/user-attachments/assets/23d9f6ef-6721-43c1-af52-5a65f7fc5a37" />


以下を実行してみるとフラグが表示される。
```
curl -X POST http://localhost:3000 -d "username=toString"
```

JavaScript のオブジェクトuserはObject.prototypeにつながっており、toString()がある。

つまりusers["toString"]はObject.prototype.toStringになり、

users["toString"]=function toString() { [native code] }というように関数になる。

このif文 if (!user || user.password !== password) について、

user = "toString"

user.password = undefined

password  = undefined

となり、user.passwordとpasswordが一致するのでフラグが返される。
