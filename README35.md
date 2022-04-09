# セクション 14: 本番環境への対応

## 98 ブラウザのサードパーティ Cookie 拒否に対応する

### 現状の問題点

Safari でリロードするとログアウトする<br>

原因：refresh_token が Cookie に保存されていない<br>

サードパーティの Cookie を拒否する仕様のため<br>

2022 年までに chrome も上記仕様になる予定<br>

### Cookie 拒否の解決方法

同一サイトとみなされれば Cookie は拒否されない<br>

同一サイトの定義は => 同じドメインのサブドメイン同士<br>

`Nuxt`udemy-v2.example.com -- `DNS` example.com -- `Rails`api.udemy-v2.example.com <br>

https://web.dev/same-site-same-origin/ <br>

### このチャプターで達成すること

| Nuxt.js（フロント）                 | Rails（サーバ）                      |
| ----------------------------------- | ------------------------------------ |
| 1 □ Heroku にカスタムドメイン設定   | 6 □ Heroku にカスタムドメイン設定    |
| 2 □ ドメインの DNS 設定             | 7 □ ドメインの DNS 設定              |
| 3 □ SSL 化                          | 8 □ SSL 化                           |
| 4 □ Rails の API_DOMAIN の値の変更  | 9 □ Nuxt.js の API_URL の値を変更    |
| 5 □ 常時 SSL 化（リダイレクト処理） | 10 □ 常時 SSL 化（リダイレクト処理） |

### このチャプターを始める前に

- カスタムドメインを取得済みであること<br>

* Heroku にクレジットカードを登録済みであること<br>

- SSL 化には、\$7 x 2app/月 の料金がかかること<br>

https://jp.heroku.com/pricing <br>

(2021 年時点)<br>

### 99 Nuxt.js にカスタムドメインを設定して SSL 化する

- `front $ heroku login`を実行<br>

* `front $ heroku domains:add www.takapro-tech.site`を実行<br>

- `front $ heroku domains`を実行して確認<br>

```
=== tk-railsnuxtv1-front Heroku Domain
tk-railsnuxtv1-front.herokuapp.com

=== tk-railsnuxtv1-front Custom Domains
Domain Name           DNS Record Type DNS Target                                              SNI Endpoint
www.takapro-tech.site CNAME           fathomless-taiga-o7ceundax1gabha0ex721nu4.herokudns.com pteranodon-17263
```

taka-project777@taka.com <br>

https://qiita.com/tksh8/items/ab3748bcb01316461abe

https://qiita.com/ozin/items/62bc7ef1dd3c827177fb <br>
