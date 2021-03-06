(*) 33,34,39,40,44問目をチェック

#### Chapter5 IPルーティングの基礎
- **内容**
  - **ルーティングの概要**
  - **ルーティングテーブル**
  - **スタティックルーティング**
  - **ダイナミックルーティング**
  - **デフォルトルート**
  - **メトリックとアドミニストレーティブディスタンス**
  - **クラスフルルーティングプロトコルとクラスレスルーティング**
  - **ディスタンスベクター概要**
  - **経路集約**
  
[コマンドメモ]

|コマンド    |説明         |
|:-----------|:------------|
|(config)#ip route <address> <mask> <next-hop>|スタティックルート設定|
  
  
  
**[ルーティング]**
パケットを宛先ホストに届けるために最適なルートを選択するプロトコル(aから順に実行する)
  1. パケットを受信するとLayer3ヘッダにある宛先IPアドレスを確認する
  2. 次にルーティングテーブルを参照してルーティングを実行する
  3. 転送先であるネクストホップアドレスのアドレスを決定
  4. 発信するインターフェースへパケットをスイッチングして転送する
    
- **スタブネットワーク**
  - 外部ネットワークへの接続が1つしかない端末のネットワークのこと
  - 管理者は対向ルータに対してスタブネットワークへの経路情報を1つだけ登録すればいいのでスタティックルーティングが適している


**[スタティックルーティングの特徴]**
  1. 管理者が決定した経路情報を手動で登録する
  2. 他のルータへ経路情報を通知しないため、セキュリティが強固
  3. 帯域幅を消費しない
  4. トポロジ変更によるルーティングテーブルの自動更新を行わない
  5. 大規模ネットワークでは管理者の手間がかかる
  6. スタブネットワークへのルーティングに最適


|種類        |コード       |
|:-----------|:------------|
|OSPF        |O            |
|RIP         |R            |
|EIGRP       |D            |
|static route|S            |
|直接ルート  |C            |


**[デフォルトルート]**
  - 特徴 
    - ルーティングテーブルにない未知の宛先へのパケットを破棄する
    - ルーティングテーブルのサイズを小さくできる
  - 用いられる理由
    - メモリやCPUなどのシステムリソースに制限がある場合
    - スタブネットワークのルータ
    - インターネット接続しているルータ
    - 特定のネットワークを学習することが望ましくないトポロジの場合
  - デフォルトルートの設定
    - `(config)#ip route 0.0.0.0 0.0.0.0 <next-hop>`
    - 0.0.0.0/0はデフォルトルートを示している


- アドミニストレーティブディスタンス
  - 1つの宛先に複数の情報源がある場合、最も優先度の高い信頼源からの情報だけをルーティングテーブルに登録する 
  - この優先度を決めるための管理値を**アドミニストレーティブディスタンス**という
  - よって経路情報の信頼性を示す尺度になる 
  
|経路の情報源|デフォルトのAD値|
|:-----------|:------------|
|直接接続         　|0     |
|スタティックルート |1     |
|EIGRP(集約)        |5     |
|BGP(外部)          |20    |
|EIGRP(内部)        |90    |
|OSPF               |110   |
|RIPv1 RIPv2        |120   |
|EIGRP(外部)        |170   |
|BGP(内部)          |200   |
|EIGRP(内部)        |C     |
|不明               |255   |

(*)AD値が低い方が優先度が高い

|プロトコル  |メトリック   |説明         |
|:-----------|:------------|:------------|
|RIPv1 RIPv2 |ホップカウント|ルータから宛先ネットワークまでに経由するルータの数|
|OSPF|1      |コスト   |インタフェースの帯域幅から算出される/管理者が手動でコスト値を決めるられる |
|EIGRP       |帯域幅と遅延|インターフェースの帯域幅と遅延から算出される値.オプションで負荷と信頼性を利用することも可能　|

- メトリックとアドミニストレーティブディスタンス(AD)の違い
  - `メトリック`:1つのルーティングプロトコルが最適経路を決定するための値
  - `AD`        :複数のソースから同じ経路情報を受け取った時の優先度
    - ex (同じ経路を通るがRIPv2で用いるかOSPFを用いるか決める) 

- 回線の速度
  - `T1`
    - 通信速度1.5Mbps
    - 最大通信速度1.544Mbps
    - 別名：DS1
  - `T2`
    - 通信速度6Mbps
    - 最大通信速度6.3124Mbps
    - 別名：DS2
  - `T3`
    - 通信速度45Mbps 
    - 最大通信速度44.73Mbps
    - 別名：DS3
  - `T4`
    - 通信速度274Mbps
    - 通信容量は274.176Mbps
    - 別名：DS4

**[ルーティングテーブルとメトリック値、見方]**

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/routing-table.png)


- ルーティングテーブルの見方(show ip route)
  - [C]は直接接続
  - Loopbackインターフェースは内部で仮想的に作られる論理インターフェースであり、物理IFではない
  - `Gateway of last resort is not set`:デフォルトルートが存在しないを意味している

[バックアップ経路の設定]
```
ダイナミックルーティングを行うとき、ルーティングプロトコルから経路情報から
失われた場合のバックアップとしてスタティックルートを選択する
ip route コマンドでスタティックルートで設定するときにAD値を付加する.
このとき、指定するAD値は使用しているルーティングプロトコルのAD値よりも大きな値にする必要がある
```

- **フローティングスタティックルート**
  - メインで使用しているプライマリ回線がダウンした場合にだけ使用されるスタッティクな経路情報のこと 
  - `ip route`コマンドの後ろにアドミニストレーティブディスタンス(AD)値を付加できる
  　-　この場合はすでに使ってるルーティングプロトコルより大きなAD値にする 

- **トラッフィク負荷分散**
  - 等コストロードバランシング
    - デフォルトで4つまでメトリックが最小の経路をルーティングテーブルに登録してLBできる 

- メモ
  - 同じ宛先ネットワークの経路情報を複数のソース(情報源)から受信した場合、アドミニストレーティブディスタンス値によって優先度を決める


- **フローティングスタティックルート**
  - メインで使用しているプライマリ回線がダウンした場合にだけ使用されるスタティックな経路情報のこと


- **自立システム(AS)**
  - 同じ運用ポリシーの元で動作するネットワークの集合のこと
  - IGP(Interior Gateway Protocol) : RIP, OSPF, EIGRP
  - EGP(Exterior Gateway Protocol) : BGP

- 等コストロードバランシング
  - Ciscoルータはデフォルトで4つまでメトリックが最小の経路をルーティングテーブルに登録してロードバランシングできる
  - OSPF,EIGRPは等コストロードバランシングをサポート
  

- **ディスタンスベクター**
  - ルーティングテーブルの情報を交換して距離(ディスタンス)と方向(ベクタ)に基づいて最適経路を決定する方式
  - どれだけの距離が必要?　どのネクストホップ(ルータ)を経由するかを基準に最適経路を選択する
  - 別名：考案者の**ベルマンフォードアルゴリズム**と呼ばれている
  - トポロジーに変化がなくても定期的にルーティングテーブルの交換を行う
  - 交換する情報量も少ないのでリソース負荷が少ない
  - 各ルーターは隣接ルータより先にあるネットワークトポロジの情報を把握していない
  - コンバージェンス(収束)速度はリンクステートよりも遅い
  - 小規模のネットワークに適している
  - 代表的には「RIP」「IGRP(現在は使われていない)」


**[ルーティングループの問題を解決しコンバージェンス(収束)を高速化する方法]**

|方法　　　　|説明|
|:-----------|:------------|
|スプリットホライズン|受信した経路情報を受信したのと同じインターフェースからは送り返さない手法|
|ルートポイズニング|リンク障害を検知したルータがメトリックを最大値(無限大)にして隣接ルータに経路がダウンしたことを通知する手法|
|ポイズンリバース|メトリックが最大の経路情報を受信すると、同じインターフェースからその経路情報メトリック最大値のままを送り返す方法.実際にはスプリットホライズンが適用されるので同じインターフェースからのアドバタイズは抑制される.ただし、メトリックが最大値の経路情報を受信した場合にはスプリットホライズンよりも優先してメトリックが最大値の経路情報を送り返す|
|ホールドダウンタイマー|ダウンした経路情報が謝って再登録されるのを防ぐための待ち時間を作る機能。ホールドダウンタイマーがセットされている期間中、その経路は『possibly down』になる.ホールドダウンタイマーが有効な期間中に、元のメトリックよりも優先度の低い(値が小大きい)経路情報を受信した場合にはそのアップデートを無視する.また、ダウン中に優先度の高いメトリックの情報がきた場合はより適切な経路情報であると判断してホールドダウンタイマーを解除してその経路情報をルーティングテーブルにセットする|
|トリガーアップデート|ディスタンスベクターはネットワークに変更がなくても定期的にアップデートを送信するが、トリガーアップデートは定期的なアップデートを待たずにネットワークの変更を検知すると即時に送信されるアップデート.これにより通常よりコンバージェンスを高速化できる.|


**RIP**(Routing Information Protocol)の特徴

|特徴　　　　|説明|
|:-----------|:------------|
|RFC標準のルーティング|RFCで定義されているマルチベンダ対応のルーティングプロトコル|
|ディスタンスベクター型|隣接ルータとテーブルを交換するダイナミックルーティング|
|メトリックはホップカウント|各宛先ネットワークへの最適経路はホップカウントに基づいて決定|
|ホップ数の上限は15|ルータ15台先(15ホップ)のネットワークまでを到達可能とする|
|30秒間でアップデート|ネットワーク状態に変更がなくても30秒感覚でマルチキャストまたはブロードキャストで送信する|
|等メトリックのロードバランシング|メトリックの等しい最適経路が複数ある場合、デフォルトでは最大4経路をルーティングテーブルに学習してロードバランシングする|

**[RIPv1とRIPv2の比較]**

|   　 　　　          |RIPv1        | RIPv2       |
|:---------------------|:------------|:------------|
|ルーティングプロトコル|クラスフル　 |クラスレス   |
|VLSMのサポート        | ×　　　　　 |○            |
|ドレッシングタイプ|ブロードキャスト(宛先IP: 255.255.255.255)|マルチキャスト(宛先IP: 224.0.0.9)|
|手動経路集約のサポート|×(自動集約のみ)|○(デフォルトで自動集約が有効)|
|認証サポート          |×            |○(MD5,テキスト認証)|
|RFC定義              |1058         |1721.1722.2453|

- クラスレスルーティングプロトコル
  - ルータ同士が交換するルーティング情報の中にサブネットマスクを含めているルーティングプロトコル

- 経路集約
  - ネットワークアドレスの共通するビット部分までプレフィックス長を変更するのでプレフィックス長の値は小さくなる 
  - 利点
    - ルーティングテーブルのサイズが小さくなる
    - ルータのメモリ使用量やCPUの負荷が減少する
    - フラッピングの影響でルーティングテーブルの更新が必要な範囲を局所化することができる
　  - フラッピングなどによるトポロジ変更が影響する範囲を小さくできる
      - 何らかの原因でインターフェースがUp,Downを繰り返している状態をいう
   
- クラスフルルーティング
  - RIPv1 IGRP
    - サブネットマスクを含まない 

- クラスレスルーティング 
  - RIPv2 EIGRP OSPF
    - サブネットマスクを含む 

- **VLSM**(Variable Length Subnet Mask)
  - 可変長サブネットマスク 
  - クラスレスルーティング
  - 同じネットワークアドレスから派生するサブネットのマスク長を可変にしてIPアドレスを割てる
- **FLSM**(Fixed Length Subnet Mask)
  - 固定長サブネットマスク 
  - 1つのネットワークアドレスから派生するサブネットのマスク長を固定して割り当てる必要がある

- スタブネットワークへ接続
  - スタティックルートを設定する
```
(config)#ip route <target_ip> <netmask> <自分のIFを指定 or next-hop_address>
```

- ARP(Address Resolution Protocol)
  - ARPはIPアドレスを基にMACアドレスを解決する 
  - ARPテーブルを表示するには`show ip arp`
  - Windowsでは`arp -a`
  - IPアドレスに基づいて相手のMACアドレスを調べる(ブロードキャスト)　: ARPリクエスト
  	 - ARPリクエストに対する返答はユニキャスト
  - ARPテーブルの「-」の記号はルータ自身のインターフェースを基に登録されている
  - 「-」が表示されているとインタフェースはアクティブな状態
  - ルータはLayer2ヘッダの送信元MACアドレスと宛先MACアドレスを変更する
     - 送信元MACアドレス: ルータ自身の出力インターフェースのMACアドレス
     - 宛先MACアドレス: 宛先IPアドレス(またはネクストホップアドレス)
  - ARPによって動的に登録されたエントリには経過時間が表示される
