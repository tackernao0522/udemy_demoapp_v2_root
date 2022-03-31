# セクション 12: サーバーサイドのログイン認証

## 71 JWT とは何か?

- Json Web Token の略<br>

2 つのパーティー間で情報を安全に送信するための方法<br>

### JWT の実態

- JSON オブジェクトをエンコードした文字列<br>

```json:sample.json
{
  "sub": "1234567890",
  ...
}
```

↓

(エンコードされた文字列) = トークンと呼ぶ<br>

### ３つに分かれるトークン

eyJhbDcioiJIUzUlNilsxxxxxxx. => ヘッダ.<br>
eyJhbDcioiJIUzUlNilsxxxxxxx. => ペイロード.<br>
eyJhbDcioiJIUzUlNilsxxxxxxx. => 署名.<br>

### ヘッダの情報

エンコード eyJhbDciOiJIUzU1NilsInR5cCI6IkpXVCJ9.<br>
デコード { "alg": "HS256", "typ": "JWT" }<br>

トークンのタイプ・署名アルゴリズム(手順・計算方法)の情報が入っている<br>

### ペイロードの情報

エンコード eyJhbDciOiJIUzU1NilsInR5cCI6IkpXVCJ9.<br>
デコード { "sub": "1234567890", "name": "John Doe", "iat": 1516239022 }<br>

任意の情報を埋め込むことができる(有効期限・ユーザー識別情報)<br>

### 署名の情報

エンコード eyJhbDciOiJIUzU1NilsInR5cCI6IkpXVCJ9.<br>

- JWT の送信者が本人であること<br>

* JWT が改ざんされてないこと<br>
  を確認するために使用される<br>

  <署名アルゴリズムが HS256 だった場合の署名作成方法><br>

HMACSHA256( base64UrlEncode(header) + "." + base64UrlEncode(payload), your-256-bit-secret )
↓
エンコードされたヘッダ + "." + エンコードされたペイロード, シークレットキー<br>
