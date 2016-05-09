### network周りの勉強用


- 不完全に設計されたネットワーク問題
 - 障害ドメイン
    - NWに問題が発生しても被害を最小限に抑えることが出来る
 - ブロードキャストドメイン
 - 宛先不明なMACアドレスを持つ大量のユニキャストトラフィック
 - 無制限のマルチキャストトラフィック


## VLAN

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
**`#switchport mode < >`** コマンド/ **vlanコマンド**

|パラメータ  |説明         |
|:-----------|:------------|
|switchport mode trunk |ポートを802.1Qへと静的に設定してトランクモードで使う|
|switchport mode access|ポートのトランクモードを無効にして非トランクで使う|
|switchport mode dynamic desirable|リンクから非トランクにする trunk, desirable,auto の場合はトランクで使う|
|switchport mode dynamic auto|リンクから非トランクにする trunk, desirable の場合はトランクで使う|
|show interface trunk|trunkモードの確認|

- vlanの作成
```
Switch#configure terminal
Switch(config)#vlan 2
Switch(config-vlan)#name test_vlan2 
```
-> `VLAN-idは 1〜1005で設定し vlan.datファイル(vlanデータベース)に保存される`

- vlanポートを割り当てる
```
Switch#configure terminal
Switch(config)#interface range fastethernet 0/2 - 4
Switch(config-if-range)#switchport access vlan 2
Switch#show vlan
```
|パラメータ  |説明         |
|:-----------|:------------|
|show vlan value-id|vlanの確認|
|show vlan name value-name|vlanの確認|
|show interface trunk|trunk状態のポートを確認|
|show vlan brief|VLANメンバシップの確認|
|show interface fa0/2 switchport|VLANメンバシップ(インターフェースf0/2)の確認|

-> 新しいVLANに再割り当てするとポートは元のVLANから自動的に削除され、新しいVLAN番号のみ適用
-> vlanを削除するとvlan1も付与されず、すべての通信が止まる.vlan1を明示的に付与するといい


##スパニングツリー

**STP(Spanning Tree Protocol)** 
```
- Layer2のリンク管理プロトコル
- スイッチネットワークにおいて不要なループを排除
- パスに冗長性を持たせる
```
- **相互接続テクノロジ**

|パラメータ  |説明         |
|:-----------|:------------|
|ファーストイーサーネット(100Mbps)|100Mbpsを提供, 10Mbpsから100Mbpsに引き上げることが可能|
|ギガビットイーサーネット(1000Mbps)|1000Mbpsを提供, 光ファイバーケーブルを対線で利用|
|ファーストイーサーネット|10ギガビットイーサネットはアップリンクでの一般使用が期待される|
|EtherChannel|2スイッチの2リンク間のLayer2に置いて帯域幅のリンク集約機能を提供する|


- **EtherChannelについて**
```
-> 標準的にはリンクアグリゲーションと呼ぶ
-> 複数のファーストイーサまたはギガビットイーサスイッチポートを単一の論理チャンネルに逆多重化する技術 
```

**EtherChannelの利点**
 - 非常に広域幅の論理リンクを作成できる
 - 関与する物理リンク間で負荷分散を行う
 - 自動フェールオーバー機能を提供
 - 以降の論理設定を簡素化する


#### 冗長トポロジ
```
-> 冗長設計により、どこか１カ所に障害が発生しても引き続き通信が継続できるようになる
-> 冗長設計によって問題が生じる事がある
```
- 冗長設計の問題
 - **ブロードキャストストーム**
   - ループ回避機能が動作していない場合、各スイッチやブリッジがブロードキャストのフラッディングを無限に繰り返す
 - **同一フレームの複数コピー**
   - ユニキャスト通信時、同一のフレームが複数コピーされて宛先ステーションへ配信される場合がある.同一フレームを複数受信するとことは想定されていないために回復不可能になる 
 - **MACアドレステーブルの不安定性** 
   - 同一のフレームを異なるポートで受信するとMACアドレステーブルの内容が不安定になる 
 

#### スパニングツリープロトコル(STP)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/stp-fig1.png)

```
1.ルートブリッジを決定
 -> ブリッジ(switch)IDが最小のスイッチが選ばれる
   ->(ブリッジID) A < B < C

2. 非ルートブリッジでルートポート(RP)を決定する
　 1. パスコストが最小のポート
　 2. (BPDU)Bridge Protocol Data Unitの送信元ブリッジが最小のポート
　 3. BPDUポートIDが最小のポート

3. 各セグメントで指定ポート(DP)を決定
 -> ルートブリッジに近いポート

4. 残りが非指定ポート(NDP)
```

#### スパニングツリーのポート状態
```
スパニングツリーが有効なNW上ですべてのブリッジは起動してから

[ブロッキング] -> [リスニング] -> [ラーニング]

と移行する.

- 適切に設定されていれば、ポートは[フォワード]か[ブロック]の状態で安定する.
- トポロジが変化している間はポートは一時的にリスニング状態かラーニング状態になる.
```
|ポートの状態|BPDU送受信   |フレーム転送 |
|:-----------|:------------|:------------|
|ブロッキング| 受信のみ    |行わない     |
|リスニング  | 送受信      |行わない     |
|ラーニング  | 送受信    |MACアドレスの学習のみ行う|
|フォワーディング| 送受信   |行う     |

※ ブロッキング(20秒) -> リスニング(15秒) -> ラーニング(15秒) -> フォワーディング
```
-> リスニング状態でルートブリッジなどを決める
-> ラーニング状態でMACアドレステーブルを作成する
```
