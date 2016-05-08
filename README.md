### network周りの勉強用


- 不完全に設計されたネットワーク問題
 - 障害ドメイン
    - NWに問題が発生しても被害を最小限に抑えることが出来る
 - ブロードキャストドメイン
 - 宛先不明なMACアドレスを持つ大量のユニキャストトラフィック
 - 無制限のマルチキャストトラフィック


### VLAN

#### VLANの概要/ビジネスグループ化
```
-> 論理的なブロードキャストドメインであり、複数の物理LANセグメントにまたがることができる
-> VLANを使用するとセグメント化を実装し、組織的な柔軟性を実現できる
-> ユーザの物理的な位置に関係なく、機能、プロジェクトチーム、アプリケーションに論理的なセブメントにグループ化できる.
```


- 階層型アドレッシング
  - 管理及びトラブルシューティングの容易性
  - エラーの最小化
  - ルーティングテーブルエントリの削減
  
- ルーティングテーブルエントリの削除
  -  ルーティングテーブルの再計算時や検索処理におけるCPUサイクルの低減
  -  ルータメモリ要件の緩和
  -  ネットワーク変更に対する迅速なコンバージェンス
  -  トラブルシューティングが容易
  
#### VLANメンバシップモード
|種類    |説明         |
|:-----------|:------------|
|スタティックVLAN|ポートに対してVLANを割り当てる|
|ダイナミックVLAN|MACアドレスに対してVLANを割り当てる|
|音声VLAN|IP電話用|


#### 802.1Q トランキング/フレーム
トランクは単一のリンク上で複数のVLANのトラフィックを伝送する.ネイティブVLANに関連付けられる
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/trunk-image.png)

802.1Qフレームは下記の図の通りにオリジナルフレームにタグフレームがつく.

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/vlantag-frame.png)

#### VTP(VLAN Trunking Protocol)
`実務ではほとんど使わないが試験には出たりする`

VTP(VLAN Trunking Protocol)はVLANの追加、削除、名前変更などのVLAN設定の一元管理を目的としたCisco独自のLayer2プロトロコル.これを利用するとVLAN名の重複やVLANタイプの不適切設定などのオペミスを最小限に抑えることができる.

- VTPモード
 - サーバ
 - トランスペアレント
 - クライアント
※ 後に説明を記載


#### 802.1Qとランキングの設定
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/network-vlan-set.png)

※画像はこちらからお借りしました(http://www.ccstudy.org/study/vlan/vlanroute2/vlanroute2.html)

**[目的]**
```
・VLAN10とVLAN20という異なるVLANに所属するパソコン同士を通信できるようにする
・トランク接続に対応したルーター（100Mポートを持つCisco2620など）を使ってVLAN間通信を実現する
```

- **スイッチにVLANを設定**
```
スイッチのFa0/1をVLAN10に設定し，Fa0/2をVLAN20に設定
これは，トランク接続機能を持たないルーターを使う場合と同じ

SwitchA(config)#interface fastEthernet 0/1
SwitchA(config-if)#switchport mode access
SwitchA(config-if)#switchport access vlan 10

SwitchA(config)#interface fastEthernet 0/2
SwitchA(config-if)#switchport mode access
SwitchA(config-if)#switchport access vlan 20
```

```
違うのは，ルーターとつながるFa0/12を，トランク接続にするところです。
こうして，スイッチがルーターにタグ付きフレームを送るようにします。

SwitchA(config)#interface fastEthernet 0/12
SwitchA(config-if)#switchport mode trunk
```

- **ルーターの設定**

次に，ルーターにサブインタフェースを作ります。
```
VLAN10を収容するサブインタフェースをFa0.1（Fa0の1番目のサブインタフェース）とします。
（ちなみに，サブインタフェース0番はメインインタフェース用に予約されているので利用できません）
「encupsulation dot1Q 10」というコマンドで，VLAN10のIEEE802.1Qのタグ付きフレームをやりとりできるようにします。
さらに，VLAN10のデフォルト・ゲートウエイになるアドレス（192.168.0.254/24）を設定します。

RouterA(config-if)#interface fastEthernet 0.1
RouterA(config-subif)#encapsulation dot1Q 10
RouterA(config-subif)#ip address 192.168.0.254 255.255.255.0

同様に，VLAN20を収容するサブインタフェースをFa0.2（Fa0の2番目のサブインタフェース）にします。

RouterA(config)#interface fastEthernet 0.2
RouterA(config-subif)#encapsulation dot1Q 20
RouterA(config-subif)#ip address 192.168.1.254 255.255.255.0
```
