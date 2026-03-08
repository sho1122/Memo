# guess.js

## 問題

https://alpacahack.com/daily/challenges/guessjs

## 解法

globalThis の SECRET を 1337 に書き換える
```
{x=globalThis[Object.keys(globalThis).find(k=>globalThis[k]===0)]=1337}
```
