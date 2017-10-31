# Web開発研修

## 目的

- Railsを通してWeb開発を学ぶ
- 開発プロセスを学ぶ
- API開発を通してマルチデバイスを意識する

この研修では[Ruby on Rails](http://rubyonrails.org/)を利用してWebアプリを開発していきます。Railsを選んだ理由としては、ペパボのサービス開発に採用されていることが大きな理由です。また、Railsの思想は他の言語のフレームワークにも影響を与えているため、最初に学ぶWebアプリケーションフレームワークとして適していると判断したためです。ただし、Rails自体を勉強することが、この研修の主たる目的ではありません。Railsを用いてWebアプリを開発する過程で、MVCやデータベースなど、Web開発に必要な知識を得てもらうことに重きを置いています。
また、ペパボではコードに変更を加えるときは、原則プルリクエストを出してレビューをしてもらいます。不明な点や相談があれば、IssueやSlack、あるいは口頭で話をして解決します。これらのような、ペパボの開発現場で当たり前に行われていることを身につけてください。

## 内容

### Rails Tutorial

[Rails Tutorial](https://railstutorial.jp/)に沿ってWebアプリを開発していってください。リポジトリは[GitHub](https://github.com/)につくり、章ごと、エクササイズごとを目安にプルリクエストをつくるようにしましょう。プルリクエストを出したらコードレビューを依頼し、OKをもらってからマージしてください。わからない点があればいつでも気軽に質問して頂いて結構ですし、お互いに助けあって研修を進めていきましょう。

### JSON API

Rails Tutorialを完走したあとは、JSON APIを追加していきます。みなさんが構築したWebアプリは、いまのところブラウザからアクセスすることを前提とした作りになっていると思います。ブラウザからアクセスする場合は、文章としての構造や装飾の重要性が高いため、HTMLやCSSがレスポンスとして返ってくることが一般的に望ましいでしょう。ですが、今後の研修でつくるモバイルアプリなどのクライアントからこのWebアプリへアクセスすることを考えると、HTMLは最適なフォーマットとは言えません。プログラムから利用しやすくするためには、より機械がパースしやすく、構造化された形式で情報を返すべきです。そのようなフォーマットとして、いくつかの仕様が策定されていますが、今回の研修では[JSON](http://www.json.org/)を採用します。

またモバイルアプリだけでなく、タブレットやスマートウォッチなど、クライアントが増え続けるマルチデバイスの世界になっている昨今では、それらの要素にアプリケーションの設計を縛られることがあります。それを解決できる方法の１つがこのAPI開発になります。APIを開発することにより、フロントエンドとバックエンドを疎結合にし、アプリケーションの機能開発と画面の装飾の関心を分離して開発できるようになります。
興味のある人は、「Web API」、「API first」などで検索してみてください。またJSON APIの他に [GraphQL](http://graphql.org/) といったものも最近話題になっています。RESTによるJSON API以外のアーキテクチャも調べてみるとよいでしょう。

#### レスポンスの構造

WebアプリをJSONを返せるように開発していくにあたり、その設計を行わなければなりません。どのようなエンドポイントにどういったリクエストを送ることで、何の情報がどのような構造で返ってくるかを決める必要があります。今回の研修では、それらを全て自由に決めて開発してください。以下に参考程度に例を示しますが、こちらから指定はしません。

なお、世の中には、[Twitter REST API](https://dev.twitter.com/rest/public)、[GitHub API](https://developer.github.com/v3/)、[カラーミーショップ API](https://shop-pro.jp/?mode=api_started)など様々なJSON APIが存在していますが、それらのレスポンス構造は統一されているわけではありません。[jsonapi.org](http://jsonapi.org/)など、共通の規格を定めようという動きはありますが、広く採用されているとは言い難い状況です。

```json
# ユーザの情報を取得する 1

GET https://example.com/api/users/1
Headers:
  Authorization: "Bearer XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  Accept: "application/json"

---

{
  "user": {
    "name": "Example User",
    "email": "example@railstutorial.org"
  }
}
```

```json
# ユーザの情報を取得する 2

GET https://example.com/api/users/1
Headers:
  Authorization: "Bearer XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  Accept: "application/json"

---

{
  "user": {
    "id": 1
    "name": "Example User",
    "icon_url": "https://secure.gravatar.com/avatar/xxxxxxxxxxxxxxxxxxx",
    "microposts": [
      {
        "content": "ハー、疲れた",
        "created_at": "2016-06-24T23:59:59+09:00"
      }
    ]
  }
}
```

```json
# 認証のためのトークンを得る

POST https://example.com/api/auth
Headers:
  Accept: "application/json"
Params:
  email: "example@railstutorial.org"
  password: "foobar"

---

{
  "user": {
    "id": 1
    "name": "Example User",
    "icon_url": "https://secure.gravatar.com/avatar/xxxxxxxxxxxxxxxxxxx",
  },
  "token": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
}
```

```json
# タイムラインを得る 1

GET https://example.com/api/feed?p=2
Headers:
  Authorization: "Bearer XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  Accept: "application/json"

---

{
  "microposts": [
    {
      "user_id": 30,
      "content": "起きた",
      "created_at": "2016-06-25T02:50:20+09:00"
    },
    {
      "user_id": 44,
      "content": "寝たい",
      "created_at": "2016-06-24T23:59:59+09:00"
    },
  ],
  "prev": "/api/feed?p=1",
  "next": "/api/feed?p=3"
}
```

```json
# タイムラインを得る 2

GET https://example.com/api/feed?since=201606250000
Headers:
  Authorization: "Bearer XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
  Accept: "application/json"

---

{
  "microposts": [
    {
      "user_id": 30,
      "content": "起きた",
      "created_at": "2016-06-25T02:50:20+09:00"
    }
  ]
}
```

#### 認証

これまではクッキーベースの方法を用いて認証を行ってきましたが、ドメインをまたぐアクセスが難しいことや多様なデバイスへの対応、フロントエンドとバックエンドの疎結合化など、いくつかの理由から、クッキーを用いた認証はJSON APIに適さないとされています([参考](https://auth0.com/blog/2014/01/07/angularjs-authentication-with-cookies-vs-token/))。代替案としてよく言及されているのは[JWT](https://tools.ietf.org/html/rfc7519)です。

##### JWT

JWTは JSON Web Token の略称で、JSONをURLセーフな表現で転送するための規格です。RFC7519で策定されています。JWTには[JWS](https://tools.ietf.org/html/rfc7515)を使ったものと、[JWE](https://tools.ietf.org/html/rfc7516)を使ったものがあります。単にJWTと言った場合はJWSを使ったものを指すことが多いため、ここでもその意味で使用します。

JWTはHeader, Payload, Signatureの3つを`.`で結合したものです。Headerは以下の様なJSONをBase64エンコードしたもので、typeと署名に用いるアルゴリズムなどが記述されます。

```json
{"type": "JWT", "alg": "HS256"}
```

Payloadも同様に、Claimと呼ばれるJSONをBase64エンコードしたものです。[いくつかのキーは予約されており](https://tools.ietf.org/html/rfc7519#section-4.1)、特殊な意味を持ちます。

```json
{"iss": "example.com", "exp": 1466839180, "user_id": 33}
```

これらを`.`で結合すると以下の文字列が得られます。

```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJleGFtcGxlLmNvbSIsImV4cCI6MTQ2NjgzOTE4MCwidXNlcl9pZCI6MzN9
```

そして、この文字列をHeaderに記述されたアルゴリズムと鍵(例として'hogehogefugafuga')を用いてサーバが署名したものがSignatureとなります。上の文字列の末尾に、`.`とSignatureを追加してJWTが得られます。

```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJleGFtcGxlLmNvbSIsImV4cCI6MTQ2NjgzOTE4MCwidXNlcl9pZCI6MzN9.x6QTuW2mqACPsotmD7dnGCXB687CJsagwX7C2bwZStc
```

つまりJWTは、`ヘッダー.ペイロード.署名されたヘッダーとペイロード`という形になっていると言えます。注意点として、前半の2つは単にBase64エンコードされた文字列であるため、このトークンを入手さえすれば誰でも中身を見たり書き換えたりすることができます。ですが、後半部の署名は鍵を知っていなければできないため、書き換えられたことを検知することが可能です。

今回のアプリでは、以下の手順を踏むことで認証することができるでしょう。

1. アプリから`email`, `password`をAPIに送る
1. 正しい組み合わせなら、`user_id`を含んだJWTを返す
1. 以降のリクエストヘッダにそのトークンを付与する

余談ですが、認証と認可は同じ意味ではありません。これらの違いを解説している文書は数多くあるので詳細はそちらに譲りますが、今回のアプリに当てはめて簡単に説明すると以下のようになります。

<dl>
  <dt>認証</dt>
  <dd>正しいemailとpasswordの組み合わせを知っていることや、サーバが発行したトークンを持っていることを根拠に、リクエストしている人がAさんであることを確かめること</dd>
  <dt>認可</dt>
  <dd>認証した上で、micropostの削除などのリソースアクセスの可否を判断すること</dd>
</dl>

### おかわり課題：JavaScriptでAPIと通信する

認証および認証が必要なアクションの実装まで終えることができたでしょうか。まだ時間に余裕があれば、おかわり課題として、JavaScriptを使ってAPIと通信してみましょう。

どの機能をJavaScriptで実装していくかは自由に設定していただいて構いませんが、思いつかなければ「トップページのマイクロポスト表示をJavaScriptで行うようにし、リロードボタンを押すと新しいマイクロポストを取得できるようにする」という課題をやってみてください。ページ遷移せずに動的に画面を書き換えるAjaxの恩恵をわかりやすく感じられると思います。

JSON API化と同様に、設計・実装方法も自由に決めて開発してください。今回はそこまで複雑な要件ではないので、jQueryだけでも十分に実装できると思います。「jQueryの一歩先を目指したい…！」という方は、ReactやVue.jsといったフレームワークの利用を検討するとよいでしょう。

以下は参考として、Webフロントエンドの歴史と、主要なフレームワークとそのアーキテクチャを軽く紹介します。

#### Webフロントエンドのリッチ化と複雑化

マルチデバイスの時代になり、多種多様なクライアントに合わせて最適なユーザー体験を提供していくことが必要になっていることは先に述べたとおりです。そして、それはWebブラウザからのアクセスも例外ではありません。Webの体験をよりリッチなものにしようと、「JavaScriptで非同期通信して、ページ遷移を伴わずに動的に画面を書き換える」という手法が考えられました。その手法の総称としてAjaxという言葉がありますが、これが生まれたのは2005年でもう10年以上前になります。
近年ではさらに一歩進めて、サーバサイドではHTMLを返さずAPIのみ提供し、JavaScriptだけですべての画面を描画するSPA (Single Page Application)というアーキテクチャも生まれました。ページ自体は単一で構成されており、ブラウザの遷移ではなくJavaScriptで必要な部分だけ画面を書き換えることによって遷移を実現していることから「Single Page」と呼ばれます。
実際のSPAの例としては、[mobile.twitter.com](https://mobile.twitter.com/)がわかりやすいでしょう。まるでモバイルアプリのような操作感を実現していることがわかるかと思います。

そうしていくと当然JavaScriptでやることが増えていき、コード量も増えてより複雑化していきます。RailsにおけるMVCのような、あるアーキテクチャに沿ってレイヤーを分けていったり、状態を管理していく必要が出てきました。フロントエンドのアーキテクチャにはMVVMやFluxなどがありますが、jQueryはそのような構造化の手段を持ちません(やりたければ自分で書くことになります)。そのため、Webフロントエンドのリッチ化と複雑化という問題に対応すべく、ReactやVue.js、AngularといったJavaScriptフレームワークが次々と登場しています。

#### 主要なフレームワークとそのアーキテクチャ

ここではReact、Vue.js、Angularの3つについて紹介します。ここで出てくるフレームワークやアーキテクチャについては、[jQueryのその先へ](https://www.slideshare.net/TsuchiKazu/jqueryweb-67010255)を見ると流れがつかみやすいと思います。

##### [React](https://facebook.github.io/react/)

最近のJavaScriptフレームワークの代表格であり、聞いたことのある人も多いのではないでしょうか。
ビュー層に特化しており、UIの各要素をコンポーネントに分割し、それらを組み合わせる形でページを作り上げていきます。
Reactが扱うUIのstate(状態)の管理にはFluxアーキテクチャがよく用いられており、それを実現するライブラリにReduxなどがあります。

##### [Vue.js](https://jp.vuejs.org/)

2013年に登場した比較的後発のフレームワークです。こちらもコンポーネント指向で、アーキテクチャはMVVMパターンに影響を受けています。
本体の機能が小さくSPAを前提としていないため、既存のアプリの一部機能を置き換えたり、少しずつ追加していくのに適しているとされています。
大規模向けの機能は周辺ライブラリで補う形となっており、小規模から大規模まで段階的に対応していける「プログレッシブフレームワーク」を標榜しています。

##### [Angular](https://angular.io/)

上2つとは違い、ビュー以外の機能も含むフルスタックなフレームワークです。HTTP通信や非同期処理、ルーティングなどの必要なライブラリが組み込まれており、開発自体に集中することができます。
Angular2では非同期処理において[Rxの概念](http://reactivex.io/rxjs/)が取り込まれるなど、アグレッシブに開発が続けられています。バージョン1系と2以降で非互換な部分が多くあるため、調べる際は注意が必要です。
