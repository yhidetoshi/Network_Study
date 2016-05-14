![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/cisco-ios-icon.jpg)


# Chapter3 Cisco IOSソフトウェアの基礎
- **内容**
  - **Cisco ISOソフトウェア操作**
  - **Ciscoルータ及びCatalystスイッチの基本設定**
  - **基本的な検証コマンド**
  - **コンフィグレーションの保存**
  - **Cisco ISO接続診断**





- Ciscoルータの初期設定
    - 必要なもの
      - PC/ターミナルソフト/コンソールポート/ロールオーバーケーブル
  - 工場出荷状態か設定のconfigファイルがない場合の表示
      - [Would you like to enter the initial configuration dialog? [yes/no]:] 
        - yesを押すと対話式の初期設定プログラムが実行される
        - ルータの初期状態のメッセージ
        - noを押すとCLIで設定できる



|コマンド    |説明         |
|:-----------|:------------|
|ctrl + shift + 6|ping tracerouteを中断する|
|tab　|補完|
| ↑ もしくは ctrl + p　|直前のコマンド|
| ↓ もしくは ctrl + n　|呼び出したコマンドを前に戻す|
| ?　|使用可能なコマンドを表示|
| #show histroy　|過去のコマンド表示|
| (config-if)#do show histroy　|過去のコマンド表示|
| (config)#hostname <name>　|ホスト名(識別情報)の設定|
| #show version　|ルータ製品のモデル名,実行中のiOS情報,インターフェースの種類と数,各種メモリサイズ,ルータ起動時からの時間など、本体の情報を表示|
| #show running-config　|iOSのバージョン含め、現在のコンフィグレーションが見れる|
| #show startup-config　|NVRAMに保存した時の情報を表示|
| #copy running-config startup-config　|現在の設定情報を保存|
| #reload　|ルータ、スイッチを再起動する|
| #delete flash　|フラッシュメモリ内のファイルを消去|
| (config-if)# no shutdown　|インターフェースの有効化|
| (config-if)# shutdown　|インターフェースの無効化|
| logging synchronous　|コマンド入力中に何かのメッセージ割り込むように表示されても、コマンドが改行された新しい行に表示される|
| no logging console　|コンソールに対するログ出力が無効化される|
| privilege level 15|ログインするといきなり特権EXECモードから始まる|
| logging console 7| コンソール上に出力するログメッセージのレベルを7以上に設定される(デバッグになる)|
| (config)#no ip domain-lookup 　|DNS機能の無効化|
| #terminal monitor 　|VTY(仮想端末)へのログ表示の有効化|
| #u all 　|undebug allの省略形. 全てのデバッグ機能の無効化|
| (config)#no ip http server 　|HTTPサーバ機能の無効化|
| #terminal history size 　|コマンド履歴数の指定|
| (config-line)#exec-timeout 　|EXECセッションのタイムアウトの無効化|
| (config-if)#description <string>　|インターフェース説明文の設定|


- **Pingコマンド**(@Cisco iOS)
  - 定義した時間内にエコー要求パケットが宛先に到着し、エコー応答を受信した場合にのみ成功と判断
  - User EXECモードまたは特権EXECモードでpingが使用できる
  - `ping <target_ip>`を`p <target_ip>`と省略できる
  - デフォルトではICMPエコー要求パケットを5つ送信し、タイムアウト時間は2秒になっている
  - ping実行時の表示記号
    - `!` : ICMPエコー応答を受信
    - `.` : ICMPエコー応答を受信できずにタイムアウトした
    - `U` : ICMP宛先到達不能(Destination Unreachable)を受信
    - `&` : TTL超過エラー
    - `M` : フラグメント化不可エラー
 
- Pingでの注意点
```
ICMPエコー要求はLayer2に渡されてカプセル化される。この時、宛先MACアドレスがARPテーブルに
キャッシュされていない場合、パケットは一時的にバッファリングされてARPリクエストを送信して
宛先MACアドレスを取得する.これよって、最初の数パケットはタイムアウトになって成功率が
100%にならないことがある.

Ping先が同じネットワーク(ブロードキャストドメイン)に存在しない時は、ARPによってデフォルトゲート
ウェイのMACアドレスを取得する必要がある
```

- 拡張Ping
  - 指定可能な主な項目
    - プロトコル(Layer3)
    - ターゲットIPアドレス
    - リピートカウント(パケット数)
    - データグラムサイズ
    - タイムアウト時間
    - 送信元アドレス(インターフェース名で指定可能)

(参考:拡張pingのターミナル)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ping-ex.png)

[iOSのCLIモード]

|モード    |プロンプト     |説明         |
|:-----------|:------------|:------------|
|ユーザEXECモード| >       |限られた一部の情報を表示、pingやtelnetを実行する程度に限られる|  
|特権EXECモード  | #       |全ての設定情報が見れる、設定の消去やコピー、デバッグ機能を有効可能、コンフィグレーションモードに移行可能|
|グローバルコンフィグレーションモード|(config)#|ホスト名の指定、デバイス全体にかかるグローバル設定を行うことができる|
|インターフェースコンフィグレーションモード|(config-if)#|インターフェースに対してIPアドレスや二重モードなどの設定を行ったり、管理的に無効化、有効化することができる|
|ラインコンフィグレーションモード|(config-line)#|コンソールやVTYポートに対して設定を行う|
|ルータコンフィグレーションモード|(config-router)#|OSPFヤEIGRPなどのルーティングプロトコルに対して設定を行う|


[一般的なエラーメッセージ]

|メッセージ    |意味         |
|:-----------|:------------|
|% Incomplete command|入力したコマンドが不完全(後ろに引数が足りない)|
|% Ambigous command: "xx"|入力した文字数が十分でないため、コマンドが識別できなかった|
|% Unknown command|コマンドが存在しない|
|% Invalid input detected at '^' maker|「^」マークが指している位置のコマンド構文が誤っている|


[インターフェースの確認]
```
show interfacesコマンド、show ip interface briefコマンドは(,)を挟んで
左側に "インターフェースの物理レベルの状態" が表示される.
これがシリアルインターフェースの場合、対向からの機器からキャリア検知信号を受信しているかを示す
右側は "データリンク層レベルの状態"を表示している
```
- インターフェースの状態(4種類)
  - ルータのインターフェースはデフォルトで無効化(shutdown)されている 
  - **Serial0/0 is up, line protocol is up** 
    - 物理層もデータリンク層も正常に動作している
    - データリンク層(Layer2)レベルの通信が可能
  - **Serial0/0 is up, line protocol is down**
    - 物理層は動作しているがデータリーンク層に問題がある
      - キープアライブがない
      - クロックレートが設定されていない
      - カプセル化タイマーが一致していない(フレーミングミス)
  - **Serial0/0 administratively down, line protocol is down**
    - shutdwonコマンドによって、インタフェースが管理的に無効になっている
  - **Serial0/0 is down, line protocol is down**
    - 物理層が正常に動作していない
        - ケーブルが接続されていない
        - ケーブルが正しくない
        - 対向のデバイス側でインターフェースが管理的に無効(shutdown)になっている
  - (*) 物理層up, データリンク層upでもあくまでLayer2の状態なので,ping(Layer3)が成功するかは不明


- インターフェスにipアドレスをふる
```
(config)#interface <interface-id>
(config-if)#ip address <ip-address> <subnetmask>

```

- インターフェースに設定したIPアドレスとサブネットマスクを確認する
```
#show running-config
#show protocols
```

- インターフェースの状態を確認する
```
show interfaces
show ip interface
show ip interface brief
show protocols
```

- 無効なコマンドを入力した場合
  - その文字列をホスト名としてIPアドレス変換しようとする
    - ネットワーク上のDNSサーバにブロードキャスト発行して問い合わせるために数秒かかる
  - これを避けるためには`no ip domain-lookup`コマンドを使用して名前解決機能を停止する
  - デフォルトではこの機能は有効になっている




