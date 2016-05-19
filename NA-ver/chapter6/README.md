#### Chapter6 VLANとVLAN間ルーティング
- **内容**
  - **VLANの概要**
  - **VLANの動作**
  - **スタティックVLANの設定と検証**
  - **トランクポートの設定と検証**
  - **VTP**
  - **VLAN間ルーティング**
  - **VLANおよびVLAN間ルーティングのトラブルシューティング**
  
**[VLANの特徴]**

|特徴    　 |説明         |
|:-----------|:------------|
|ブロードキャストドメインの分割|ブロードキャストドメインの分割によって、ブロードキャストドメインのサイズが小さくなり、ノード間の通信効率が高くなる|
|論理的なセグメント(グループ)を構成|VLAN単位で論理的ネットワークを分割し、ユーザを部門別にグループ化することができる|
|セキュリティの強化|トラフィックをLayer2レベルで分離できるため、異なるVLANに関わるネットワークセキュリティを強化できる|
|柔軟なネットワーク設計|物理的な接続とは関係なくユーザをネットワークに追加したり、接続変更せずに別のネットワークに分離できたりする|

- Point
  - VLANよってIPが節約されるわけではない
  - 普通の状態よりVLANの方が流れが複雑になる
  - ブロードキャストドメインの数は増加してサイズは縮小
  - コリジョンドメインへの影響はない

- ポートセキュリティ
  - 許可していないMACアドレスを持つコンピュータがスイッチポートに接続されているのを防ぐ 

**[VLANの作成]**
```
(config)#vlan <vlan-id>
(config-vlan)#name <vlan-name>
```

- **VLAN IDの範囲**

|VLAN-ID     |範囲         |説明
|:-----------|:------------|:------------|
|0,4095      |予約         |システム内部で使用|
|1           |標準         |シスコのイーサネットのデフォルトVLAN|
|2〜1001     |標準         |イーサネット用の標準VLAN|
|1002〜1005  |標準         |FDDIおよびトークンリング用のデフォルトVLAN|
|1006〜4094  |拡張         |イーサネット用の拡張VLAN|

  - デフォルトVLAN
    - 1/1002〜1005の5つ
      - デフォルトVLANは変更も削除もできない 
  - VLANの追加/変更/削除できるVLAN
    - 2〜1001/1006〜4094 
  

- スイッチポート
  - アクセスポート
    - 1つのVLANトラフィックのみを転送する
    - エンドユーザのコンピュータやサーバを接続する
  - トランクポート
    - 複数のVLANトラフィックを転送する 

- スイッチポートにVLANを割り当てる説明
  - コマンド：`(config-if)#switchport access vlan <vlan-id>`
  - VLANを作成した後にスイッチポートにVLANを割り当てるのが一般的
  - `switchport access vlanコマンド`で作成していないVLAN IDを指定した場合は
    - `「%Access VLAN does not exist. Creating vlan <vlan ID>」`と言うメッセージが表示されてVLANが作成される
    - 作成されたVLANはフラッシュメモリ内のvlan.datファイルに追加される

(ex)12ポートを持つCatalystスイッチに3つのVLANを設定
- スイッチはポートごとにコリジョンドメインを分割する
- VLANごとにブロードキャストドメインを分割する

=> コリジョンドメイン数は12, ブロードキャストドメイン数は3


- **スイッチポートに割り当てられているVLANを確認するコマンド**
  - `show vlan`
  - `show interfaces switchport`
  - `show running-config`
  - `show interface status`

**トランクの設定には下記の1,2が必要**

1.トランクプロトコルの設定
  - `(config-if)#switchport trunk encapsulation{ isl | dot1q | negotiate }` 
    - `isl` : トランクプロトコルをISLにする
    - `dot1q` : トランクプロトコルをIEEE802.1Qにする
    - `negotiate` : 両端でネゴシエーションし、トランクプロトコルを決定する(デフォルト)
    (*) ISLはCisco独自 IEEE 802.1Qは標準

2.トランクポートの設定
  - `(config-if)#switchport mode trunk` 

よってまとめると
```
(config)#interface fastethernet 0/1
(config-if)#switchport trunk encapsulation dot1q
(config-if)#switchport mode trunk
````

- **DTP**(Dynamic Trunking Protocol)
  - 対向機器のポートネゴシエーションを行って動的にトランクポートを設定するプロトコル 
  - 対向機器のポートとネゴシエーションを行い、その設定に応じてスイッチポートのタイプ(アクセスポート/トランクポート)、および、トランクプロトコル(IEEE 802.1QかISL)を動的に着替えるCisco独自の技術


**スイッチポートのネゴシエーション設定**
- `(config-if)#switchport mode {dynamic desirable | dynamic auto}`
  - `dynamic desirable`: DTPフレームを送受信して積極的にトランクポートになるようにネゴシエーションする.対向がaccessだとaccessポートになる
  - `dynamic auto`: DTPフレームを受診して対向からトランクになるように働きかけられるとトランクポートになる.対向がaccessまたはdynamic autoの場合はaccessポートになる


**明示的にトランクポートまたはアクセスポートにしたい場合のコマンド**
```
(config-if)#switchport mode { access | trunk }
```

**[ポートのリンク状態]**

|                 |access       |trunk        |dynamic desirable|dynamic auto |
|:----------------|:------------|:------------|:----------------|:------------|
|access           |access       |制限付き     |**access**       |**access**   |
|trunk            |制限付き     |trunk        |trunk            |trunk        |
|dynamic desirable|**access**   |trunk        |trunk            |trunk        |
|dynamic auto     |**access**   |trunk        |trunk            |**access**   |


- **VTP**(VLAN Trunking Protocol)
  - トランクリンクで接続された複数のスイッチ間でVLAN情報を共有するCisco独自のプロトコル
  - VLANが複数のスイッチにまたがって構成されている場合
    - VTP無し：各スイッチで同じVLAN情報を管理する必要がある
    - VTP有り：VLAN設定を1台のスイッチ上だけで実行すれば済むので整合性も取りやすい
  - 3つのモードがある
    - サーバ/クライアント/トランスペアレント 

**[VTPモード設定]**
```
(config)#vtp mode {server | client | transparent }
```
|                      |サーバ       |クライアント |トランスペアレント|
|:---------------------|:------------|:------------|:-----------------|
|VLANの作成・変更・削除| ○      　　 |   ×         |○                 |
|VTPアドバタイズの転送 | ○      　　 |   ○         |○                 |
|VLAN設定の同期        | ○      　　 |   ○         |×                 |


**トランクプロトコル**

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ether-frame.png)
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ether-frame-8021q.png)
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ether-frame-isl.png)

(図の引用：http://bit.ly/1U0p1n6)

|トランクプロトコル|IEEE 802.1Q  |ISL          |
|:-----------------|:------------|:------------|
|規格              |IEEE         |シスコ独自   |
|方式              |タギング     |カプセル化   |
|ネイティブVLAN    |サポート     |なし         |
|フレームの増加サイズ|4バイト    |30バイト(ヘッダ:26,FCS:4)|
|FCSの処理         |再計算       |フィールド追加|
|VLAN ID           |12bit        |10bit         |
|VLAN数            |4096(2^12)   |1024(2^10)    |


- UTPクロスケーブル
  - 2台のスイッチ間を接続
  - 複数のVLANを通す時も同様

- DTPネゴシエーションを使わずに固定でトランクポートに設定するコマンド
  - 固定でtrunkポート: `switchport mode trunk` 
  - DTPの無効化: `(config-if)#switchport nonegotiate`
  - 無効化されたDTPを有効化: `(config-if)#no switchport nonegotiate`

- スイッチポートがトランクとして動作していることを確認するコマンド
  - `show interface switchport`
  - `show vlan`
  - `show interfaces trunk`
  - `show interfaces status`

- `show vlan`の結果
  - デフォルトでは"VLAN0020"のように名前がつけられる
  - トランクポートは表示されない
  - VLAN1002〜1005はデフォルトのVLANで削除できない
