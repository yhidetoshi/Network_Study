![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/network-image.jpg)

### network周りの勉強用


- 不完全に設計されたネットワーク問題
 - 障害ドメイン
    - NWに問題が発生しても被害を最小限に抑えることが出来る
 - ブロードキャストドメイン
 - 宛先不明なMACアドレスを持つ大量のユニキャストトラフィック
 - 無制限のマルチキャストトラフィック


## VLAN

##### VLANの概要/ビジネスグループ化
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

※画像引用(http://bit.ly/23AIDUq)

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

|種類  |説明         |
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


##### 冗長トポロジ
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
 

##### スパニングツリープロトコル(STP)

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

##### スパニングツリーのポート状態
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

(参考)
ブロッキングからフォワーディングに移行かかる時間
NDPを持つスイッチから見て
[直接障害]30秒(リスニングからラーニングに15秒、ラーニングからフォワーディングで15秒)
[関節障害]50秒(ブロッキングに20秒 + リスニングからラーニングからフォワーディング)

```
-> リスニング状態でルートブリッジなどを決める
-> ラーニング状態でMACアドレステーブルを作成する
```

##### PortFast
```
PortFastは
Layer2アクセスポートとして設定されたインターフェースをブロッキング状態から直ちにフォワーディング状態に移行させ、リスニング状態とラーニング状態を省力する.

-> Portfastはスパニングツリーのコンバージェンスのためにアクセスポートが待機する時間を最小限にすることを目的にしている. 使用は"アクセスポートに限る"
```
**(注意: どのポートでもportfastを設定できるが、スイッチと接続するポートで設定しないように)**

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/port-fast.png)

(画像引用:http://bit.ly/23AI9h3)

##### スパニングツリーパスコスト

|リンク速度  |コスト       |
|:-----------|:------------|
|10 Gps| 2    |
|1 Gps| 4    |
|100 Mps| 19    |
|10 Mps| 100    |


## ルーティング
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/igp-egp-image.jpg)
(画像引用：http://bit.ly/1T6qb6x)
- IGP(Interior Gateway Protocol)
 - AS内でルーティング情報を交換するために使われるプロトコル
    - RIPv2 EIGRP OSPF 
- EGP(Exterior Gateway Protocol)
 - AS間で経路情報を交換するために使われるプロトコル
    - BGP

##### ルーティングプロトコルのクラス

- **ディスタンスベクター**
 - ディスタンスベクター型のプロトコルは送信先ネットワークへの距離(ディスタンス)と方向(ベクター)を表す情報を交換して経路選択を行う

- **リンクステート**
 - リンクステート型のプロトコルは、各ルータが持つリンク状態を表す情報を交換し、ネットワークトポロジをSPF(Shortest Path First)アルゴリズムを使って計算して経路選択を行う

- **拡張ディスタンスベクター**
 - ディスタンスベクターとリンクステートの利点を組み合わせたプロトコル
  
##### メトリックを使用した最短経路の選択

|種類  |説明       |
|:-----------|:------------|
|ホップカウント| ネットワーク到達するために通過するルータ数|
|帯域幅|リンクのデータ容量 |
|遅延|パケットが各ルータインターフェースを通過するために必要な時間|
|負荷|回線の帯域幅に対するトラフィック量の割合|
|信頼性|ネットワークリンクのビットエラー|
|コスト|ルータ上で管理者が設定するパラメータ値|

##### アドミニストレーティブディスタンス
```
->ルータ内にネットワークに関する経路情報が複数ある場合、アドミニストレーティブディスタンス値を使って各ルーティング情報元の信頼性を評価する.

値は[0 - 255]
```
(例)
```
OSPFのアドバタイズ値が110、EIGRPのアドバタイズ値が90 の場合
-> EIGRPの方が信頼性が高いとなる
```

|ルート情報のソース  |デフォルトのディスタンス値|
|:-------------------|:------------------------------|
|直接接続されたNW|0|
|スタティックルート|1|
|EIGRP|90|
|OSPF|110|
|RIPv2|120|
|外部EIGRP|170|
|不明信頼できない|255|

->`値が小さいほど信頼される`

#### RIP

- *RIP*
 - コンバージェンスに時間がかかる
 - コンバージェンスが遅いため一貫性のないルーティングが生成される
 - メトリックとして最大15ホップまで

[RIPのルーティングループの防止策]

|名前|説明|
|:---------------|:-----------------|
|スプリットホライズン|経路情報はそれを学習したIF側に送信しない|
|ルートポイズニング|障害経路をmetric16で通知する|
|ポイズンリバース|ルートポイズニング受信に対する返事|
|ホールドダウンタイム|ルートがダウンしたと思われる際に使うタイマ|
|トリガーアップデート|変更をトリガーにしてアップデートを送信する|

#### OSPF (リンクステートルーティングプロトコル)

1. LSA(Link State Advertisement)情報を集める
2. 集めたLSAからLSDB(リンクステートデータベース=トポロジーデータベース)
3. SPFツリーの作成
-> ネットワーク全体を把握して経路を作る/コンバージェンスが非常に高速

[リンクステートルーティングの利点と欠点]
- 利点
 - 高速コンバージェンス
 - ループフリー
- 欠点
 - 多くのルータのリソースを必要とする 


#### VLSM(Variable-Length Subnet Mask)


### EIGRP

#### EIGRPの機能

```
EIGRPはリンクステート及びディスタンスベクターの両ルーティングプロトコルの利点を組み合わせたCisco独自の拡張ディスタンスベクタールーティングプロトコル
```
|機能|説明|
|:---------------|:-----------------|
|高速コンバージェンス|DUAL(Diffusing Update Algorithm)を利用してOSPFより高速にコンバージェンスが可能|
|帯域幅使用の減少|EIGRPは定期的にアップデートせず、差分が生じた時にアップデートする|
|複数のネットワーク層サポート|PDM(Protocol-Dependent Module)を使ってApple Talk,IPv4,IPv6, Novell IPXをサポート|
|クラスレスルーティング|宛先ネットワーク毎にサブネットマスクをアドバタイズするのでVLSMをサポートできる|
|オーバーヘッドの減少|ブロードキャストではなく、マルチキャストとユニキャストを使用するのでエンドステーションはルーティングアップデートによる影響を受けない|
|ロードバランシング|不等メトリックロードバランシングをサポートするので、ネットワークのトラフィックフローをより適切に分散させることができる|
|簡単な集約|ネットワーク内の任意の場所で集約経路を作成できる|

-> デメリットはCisco独自の機能のため、**マルチベンダー環境では使用不可能**

#### EIGRPテーブル

- EIGRPは3つのテーブルを使用
 - ネイバーテーブル
 - トポロジーテーブル
 - ルーティングテーブル

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/eigrp_tables.png)

- EIGRPは宛先までの**最短経路(サクセサ)**および**バックアップ経路(フィージブルサクセサ)**を決定するために以下の2つのパラーメータを使用する.
 - アドバタイズディスタンス(AD)
   - EIGRPネイバーが特定のネットワークに到達するためのEIGRPメトリック
 - フィージブルディスタンス(FD)
   - あるEIGRPネイバーから学習した特定のネットワークのADとそのネイバーに到達するためのEIGRPの合計 

#### EIGRPの設定と確認
```
- router eigrpコマンド、networkコマンドを使ってEIGRPの設定をする
- ルーティング情報を相互に交換するすべてのEIGRPルータには同じAS番号を使用する必要がある
```

|コマンド|説明|
|:---------------|:-----------------|
|router eigrp 100|AS100のEIGRPルーティングプロセスを有効にします|
|network 172.16.0.0|ネットワーク172.16.0.0をEIGRPにルーティングプロセスに関連付ける|
|network 10.0.0.0|ネットワーク10.0.0.0をEIGRPルーティングプロセスに関連付ける|

下記の図は参考例

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/eigrp-command1.png)

#### EIGRPの経路自動集約設定と解除

- デフォルトでは,EIGRPは主要なネットワークでアドレスをクラス単位に集約するため、不連続サブネットサポートできない
  - 不連続なネットワークを使用している場合、自動集約を無効にしてルータの混乱を最小限に抑える必要がある  

- 自動集約を無効にする場合、EIGRPルータコンフィグレーションモードで`no auto-summary`コマンドを使う

#### EIGRP設定の確認

コマンドメモをテーブルに

|コマンド|説明|
|:---------------|:-----------------|
|#show ip route eigrp|ルーティングテーブル中の現在のEIGRPエントリを表示|
|#show ip protools|アクティブなプロセスのパラメータと現在の状態を表示|
|#show ip eigrp interfaces|EIGRPに設定されたインターフェースに関する情報|
|#show ip eigrp neighbors [detail]|IP EIGRPが検出したネイバーを表示|
|#show ip eigrp topology [all-links]|IP EIGRPトポロジーを表示|
|#show ip eigrp traffic |IP EIGRPパケットの数を表示|

##### EIGRPのロードバランス

- EIGRPがデフォルトでメトリックの計算で使用する基準
 - 帯域幅
   - 送信元と宛先の最小帯域幅
 - 遅延 
   - 経路上の累積インターフェース遅延 

- EIGRPでメトリックの計算時に使用する設定可能なオプション基準
 - 信頼性
   - キープアライブに基づく発信元と宛先の最悪条件の信頼性を表す 
 - 負荷 
   - パケットレートとインターフェースの設定済み帯域幅に基づいて計算された、送信元と宛先のリンクの最悪条件の負荷


##### 等コストパスロードバランシング

- デフォルトでは,EIGRPは等メトリックロードバランシングを実行
 - デフォルトでは,メトリックが最小メトリックに等しい経路が4つまでルーティングテーブルに設定される
- 同じ宛先に対して,ルーティングテーブルに最大16のエントリを格納出来る
 - エントリ数は`maxlmun-paths`コマンドで設定可能 

##### 不等コストパスロードバランシングの設定

```
[コマンド]
Router(config-router)# variance multiplier

[説明]
- その宛先への最小メトリックを持つルートにmultiplier値(1〜128)を乗じた結果よりも小さい
  メトリックの経路で、ルータによるロードシェアリングが行なわれる
- デフォルトのvariance値は1  
```

#### varianceの例

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/eigrp-variance-lb.jpg)

#### EIGRPの認証/MD5

(認証について)
- MD5認証をサポートする
- ルータあh送信するすべてのEIGRPパケットについて自己を識別する
- ルータは受診する各ルーティングアップデートパケットの送信元を認証する
- 参加ネイバーはそれぞれ、同じ鍵を設定している必要がある
 
(MD5認証の設定ステップ)

1. キーチェーン(使用可能な鍵[パスワード]のグループ)を作成する
2. 鍵IDを各鍵に割り当てる
3. 鍵を識別する
4. (オプション)鍵が有効な期間を指定する
5. インターフェースでMD5認証を有効にする
6. インターフェースで使用されるキーチャーンを指定する


## ACL(Access Control List)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/acl-icon.jpeg)

- フィルタリング
 - ルータを経由するパケットをフィルタリングしてIPトラフィックを管理する
   - 受送：インバウンド
   - 送信：アウトバウンド
- 分類
 - 特別な処理を行うトラフィックを識別する

#### フィルタリング

- 指定したルータインターフェース間で送受信されるパケット、及びルータを経由するトラフィック
- ルータを管理するためにルータのvtyポートで送受信されるTelnettラフィック


(vtyポートについて)
```
VTY line (Virtual Terminal Line)は仮想端末のライン.
コンソールポートとは異なり、実際に物理的にポートを持っている訳ではない.
Ciscoルータ上で仮想的に複数ポートを持つことにより、複数のユーザが同時にアクセスすることが可能.
デフォルトではVTYポート番号はline vty 0 4とあることから仮想ポートとしては 0～4 の5ポートで5つのtelnetセッションを接続することができる.
VTYポートは、小さい番号から使用されていきます。
```

- ルータがパケット破棄する場合、一部のプロトコルは宛先が到達不可能であることを通知する特殊パケットを返す
 - ping: `Destination unreachable(U.U.U.)`
 - traceroute: `Administratively prohibited(!A * !A)`


#### ACLのガイドライン

- 標準ACLと拡張ACLでフィルタリングできるものが異なる
- 1つのプロトコル,1つの方向、1つのインターフェースについて使用できるのでは1つのACLのみ
- ACL文の順番でテストが行なわれるため、最も特殊な文をリストの一番上にする
- 最後のACLテストは常に、残りすべてを拒否する暗黙のdeny文であるため、どのリストにも少なくても１つは`permit`文が必要
- ACLはグローバルに作成してからインバウンドまたはアウトバウンドトラフィック用にインターフェースに適用する
- ACLは適用する方法によって、ルータ経由するトラフィックをフィルタリングできるほか、ルータの出入りするトラフィックをフィルタリングできる
- ネットワークにACLを設置する場合
 - 拡張ACLは送信元近くに配置する
 - 標準ACLは宛先近くに配置する


#### その他のACL

##### ダイナミックACL

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/dynamic-acl.png)

- **ダイナミックACL(ロックアンドキー)**
 - ルータを経由しようとするユーザは、ルータにTelnetで接続して認証されない限り、ブロックされる 
 
- **利用ケース**
 - インターネットを介してリモートホストから接続したい場合
 - ネットワーク内のホストへアクセス許可をしたい場合
 - ロックアンドキーでユーザを認証し、一定期間、ファイアーウォールルータを介したホストやサブネットへの限定的なアクセスをユーザに許可する
 - ローカルネットワーク上のホストの一部だけに対していファイアーウォールで保護されているリモートネットワークのホストへ許可したい場合
 
- ダイナミックACLの利点
 - 各ユーザの認証にチャレンジメカニズムを使用
 - 大規模なインターネットワークの管理が簡素化
 - ACLに必要なルーティングプロセスの量が少なくなる
 - ハッカーによるネットワークへの侵入機会が減る
 - 他に設定されているセキュリティ制限を犠牲にせずにファイアーウォールによるダイナミックユーザアクセスが可能


#### 再帰ACL

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/reflexive-acl.png)

再帰ACL:ルータ内部から開始されたセッションに対してアウトバウンドトラフィックを許可し、インバウンドトラフィックを制限する

-> `外部ネットワークから許可されるトラフィックは内部ネットワークから発信された戻りのトラフィックだけ`

- 利点
 - ファイアーウォール防御機能に含まれる
 - スプーフィングや特定のDos攻撃に対するセキュリティ機能を提供
 

#### 時間ベースACL

- 時間ベールACL
 - 日や週などの時間単位に基づいてアクセスコントロールできる 

#### ワイルドカードマスク

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/wildcard-acl.png)

(例)ACLの条件文で「172.16.1.0/24」のネットワークを指定したい場合
```
ACLの条件文では「172.16.1.0 0.0.0.255」と入力。「172.16.1.」まではチェックをしたいので、ワイルド
カードマスクではその部分を「0.0.0.」として、第4オクテットはチェックしたくはないので、ワイルドカードマスクのビットを全て1にします. 第4オクテットのビットを全て「1」にすると10進数では 255 になります.
```
**(ルール)**
- 0は対応するアドレスビットの値と一致することを表す
- 1は対応するアドレスビットの値を無視することを表す

**(ワイルドカードマスクの省略形)**

- すべてのアドレスビットに一致させる場合
`172.30.16.29 0.0.0.0` => `host 172.30.16.29`

- すべてのアドレスビットを無視する場合
`0.0.0.0 255.255.255.255` => `any`

#### ACLの設定例

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/acl-setting1.png)

(引用:http://bit.ly/1Np2TGL)


## VPN

#### VPNについて
- VPN(Virtual Private Network)
 - プライベートネットワーク内の情報をパブリックネットワークを介して伝送する
 - トラフィックを暗号化してデータの機密性を保持する

- VPNの利点
 - コスト削減
   - 高価なWANの接続の必要がなくなる
 - セキュリティ
   - データの暗号化と認証プロトコルを使用
 - スケーラビリティ
   - 複数のISPやデバイスで構成されるインターネットインフラストラクチャの使用が可能
 - 広帯域テクノロジとの親和性
   - モバイルワーカ、在宅勤務など接続が可能
   
- VPNのタイプ
 - サイト間
   - 一般的なWANの拡張形
     - 通常のTCP/IPトラフィックをVPNゲートウェイを介して接続
 - リモートアクセス
   - Easy VPN
     - Cisco Easy VPN Server
     - Cisco Easy VPN Remote
   - SSL VPN(Web VPN)
     - リモートアクセスの２種類にはCisco VPN Clientソフトウェアを使用
     - Cisco IOS SSLベースのVPN(Web VPN)はインターネットを使用できる場所ならどこでもWebから接続可能

