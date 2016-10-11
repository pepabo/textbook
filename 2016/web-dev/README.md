# Web開発研修

## 目的

- Railsを通してWeb開発を学ぶ
- 開発プロセスを学ぶ

この研修では[Ruby on Rails](http://rubyonrails.org/)を利用してWebアプリを開発していきます。Railsを選んだ理由としては、ペパボのサービス開発に採用されていることが大きな理由です。また、Railsの思想は他の言語のフレームワークにも影響を与えているため、最初に学ぶWebアプリケーションフレームワークとして適していると判断したためです。ただし、Rails自体を勉強することが、この研修の主たる目的ではありません。Railsを用いてWebアプリを開発する過程で、MVCのやデータベースなど、Web開発に必要な知識を得てもらうことに重きを置いています。
また、ペパボではコードに変更を加えるときは、原則プルリクエストを出してレビューをしてもらいます。不明な点や相談があれば、issueやslackで話をして解決します。これらのような、ペパボの開発現場で当たり前に行われていることを身につけてください。

## 内容

### Rails Tutorial

[Rails Tutorial](https://www.railstutorial.org/book/)に沿ってWebアプリを開発していってください。リポジトリは[GitHub](https://github.com/)につくり、章ごと、エクササイズごとを目安にプルリクエストをつくるようにしましょう。プルリクエストを出したらコードレビューを依頼し、OKをもらってからマージしてください。わからない点があればいつでも気軽に質問して頂いて結構ですし、お互いに助けあって研修を進めていきましょう。

#### baytへの画像アップロード

[Rails Tutorialの11章](https://www.railstutorial.org/book/user_microposts)では、Micropostに画像を添付できるようにしていく中で、
production環境では[Amazon S3へアップロードする](https://www.railstutorial.org/book/user_microposts#sec-image_upload_in_production)ように指示されていますが、この研修ではアップロード先としてbaytを利用します。baytは[mogilefs](https://github.com/mogilefs)をバックエンドとするペパボのオブジェクトストレージで、実際に30days Album、goope、カラーミーといったサービスで日々利用されています。今回の研修ではbaytの`sandbox`バケットへのアップロードを行うため、Rails Tutorialの内容を一部読み替えて、以下の設定を行ってください。

```ruby
# Gemfile

gem 'carrierwave',     '0.10.0'
gem 'mini_magick',     '3.8.0'
gem 'carrierwave-aws', '1.0.1'
```

```ruby
# app/uploaders/picture_uploader.rb

class PictureUploader < CarrierWave::Uploader::Base
  if Rails.env.production?
    storage :aws
  else
    storage :file
  end

  def store_dir
    if Rails.env.production?
      "training/joe_noh/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"  # joe_nohという名前は適当に変更してください
    else
      "uploads/#{model.class.to_s.underscore}/#{mounted_as}/#{model.id}"
    end
  end
end
```

```ruby
# config/initializers/carrierwave.rb

if Rails.env.production?
  CarrierWave.configure do |config|
    config.aws_credentials = {
      access_key_id:     ENV['BAYT_KEY'],
      secret_access_key: ENV['BAYT_SECRET'],
      region:            'us-east-1',
      endpoint:          ENV['BAYT_URL']
    }

    config.asset_host = ENV['BAYT_ASSET_URL']
    config.aws_bucket = 'sandbox'
    config.aws_acl    = 'public-read'
  end
end
```

チュートリアルでも触れられていますが、外部に知られたくないキーなどはハードコードするべきではありません。baytへアクセスするために必要な情報も、環境変数から読み取るようにしましょう。

### JSON API

Rails Tutorialを完走したあとは、JSON APIを追加していきます。みなさんが構築したWebアプリは、いまのところブラウザからアクセスすることを前提とした作りになっていると思います。ブラウザからアクセスする場合は、文章としての構造や装飾の重要性が高いため、HTMLやCSSがレスポンスとして返ってくることが一般的に望ましいでしょう。ですが、今後の研修でつくるモバイルアプリなどのクライアントからこのWebアプリへアクセスすることを考えると、HTMLは最適なフォーマットとは言えません。プログラムから利用しやすくするためには、より機械がパースしやすく、構造化された形式で情報を返すべきです。そのようなフォーマットとして、いくつかの仕様が策定されていますが、今回の研修では[JSON](http://www.json.org/)を採用します。

#### レスポンスの構造

WebアプリをJSONを返せるように開発していくにあたり、その設計を行わなければなりません。どのようなエンドポイントにどういったリクエストを送ることで、何の情報がどのような構造で返ってくるかを決める必要があります。今回の研修では、それらを全て自由に決めて開発してください。以下に参考程度に例を示しますが、こちらから指定はしません。

なお、世の中には、[Twitter REST API](https://dev.twitter.com/rest/public)、[GitHub API](https://developer.github.com/v3/)、[カラーミーショップ API](https://shop-pro.jp/?mode=api_started)など様々なJSON APIが存在していますが、それらのレスポンス構造は統一されているわけではありません。[jsonapi.org](http://jsonapi.org/)など、共通の規格を定めようという動きはありますが、広く採用されているとは言い難い状況です。

```
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

```
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

```
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

```
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

```
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

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJleGFtcGxlLmNvbSIsImV4cCI6MTQ2NjgzOTE4MCwidXNlcl9pZCI6MzN9
```

そして、この文字列をHeaderに記述されたアルゴリズムと鍵(例として'hogehogefugafuga')を用いてサーバが署名したものがSignatureとなります。上の文字列の末尾に、`.`とSignatureを追加してJWTが得られます。

```
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
