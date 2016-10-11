# 監視

サーバに障害が起きたことや、負荷が増加していることを検知して対策するために、サーバの状態を監視します。サーバ監視ツールとして、[Zabbix](http://www.zabbix.com/)、[Nagios](https://www.nagios.org/)、[Munin](http://munin-monitoring.org/)などがありますが、ペパボでは、はてなが運営する[Mackerel](https://mackerel.io)という監視サービスを主に利用しています。この研修でも、Mackerelを使って各種メトリクスが取れることを確認してもらいます。注意にも書きましたが、Mackerelは有料のサービスですので、導入したまま放置することの無いようにお願いします。

## 手順

### Mackerelエージェント

Mackerelエージェントを導入するマニフェストを書き、踏み台サーバも含めた全台にインストールしてください。

### メトリクスの確認

ペパボのメールアドレスでMackerelにサインアップし、[ダッシュボード](https://mackerel.io/orgs/pepabo/dashboard)でグラフなどが見れることを確認してください。
