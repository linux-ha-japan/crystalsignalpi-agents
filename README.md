(Japanese)

# Crystal Signal Pi 用リソースエージェント

Copyright (c) 2019 Linux-HA Japan Project

Pacemaker のクラスタの状態を、 Crystal Signal Pi の色を変化させることで
視覚的にわかりやすく監視するためのリソースエージェントです。

Crystal Signal Pi は株式会社インフィニットループの製品です。
* http://crystal-signal.com/

## こんなことができます

* サービスが稼働してるノードに合わせて Crystal Signal Pi の色が変わります。
  * 例: ノード1では緑、ノード2では青、など
* 片系運用中(一方のノードが故障してこれ以上フェイルオーバができない場合)は、Crystal Signal Pi の色が点滅します。
* 運用停止時は Crystal Signal Pi は赤く点灯します。
  * 注: 全ての故障時に赤く点灯するわけではありません。下記留意点も参照してください。

## 使い方

* (0) Pacemaker クラスタの構築、サービスに必要なcrmリソース設定を全て完了した上で、以下を実行してください。

* (1) 全てのクラスタノード上に、下記のリソースエージェントスクリプトを配置して実行権を付与してください。

```
# cp -p agents/CrystalSignalPi /usr/lib/ocf/resource.d/heartbeat/
# chmod 755 /usr/lib/ocf/resource.d/heartbeat/CrystalSignalPi
```

* (2) crmリソース設定に以下を追加で設定してください。ここで以下の箇所は環境に合わせて変更してください。
  * `ipパラメタ`: Crystal Signal Pi のIPアドレス
  * `service-group`: 監視対象とするサービスのリソースID

```
primitive CrystalSignalPi ocf:heartbeat:CrystalSignalPi \
        params ip="192.168.100.103" \
	op monitor interval="10s" timeout="60s" on-fail="restart"

colocation colocation_CrystalSignalPi inf: CrystalSignalPi service-group
order order_CrystalSignalPi inf: service-group CrystalSignalPi
```

## パラメタ一覧

|パラメタ名 | 必須 |説明 |
|---------|:---:|------|
|`ip`     |  ◯  | Crystal Signal Pi のIPアドレス |
|`node_color_map` | - | ノード毎の色のリスト。RGB値を空白区切りのリストで指定する。クラスタを構成するノード数分の要素を指定する。<br>デフォルト値: "0,255,0 0,0,255" (緑、青の2ノード構成)|
|`blink_period` | - | 片系運用時(フェイルオーバが不可の状態時)に点滅する速度(ミリ秒)。<br>デフォルト値: 500 |
|`state` | - | リソースエージェント起動状態管理用ファイルのパス。通常変更不要。<br>デフォルト値: /var/run/resource-agents/CrystalSignalPi-RESOURCE_ID.state|

## 留意点

* 運用停止時は赤く点灯しますが、全ての故障時に赤く点灯するわけではありません。
  * 具体的には以下のような事象でサービス停止した場合には赤く点灯しない場合があります。
    * 電源断などノード故障時のフェイルオーバ中に処理が失敗し、サービス停止に至った場合
    * 片系運用中のノードが電源断などの故障でサービス停止に至った場合
  * 内部動作としては、リソースエージェントの stop 処理が実行できない場合が該当します。しょせんデモ用に作成した簡易なものということでご容赦ください…

以上
