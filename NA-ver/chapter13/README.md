#### Chapter13 ネットワークデバイスの管理とセキュリティ
- **内容**
  - **パスワードによる管理アクセスの保護**
  - **SSHの設定**
  - **VTYアクセス制御**
  - **CDP**
  - **IOSによるTelnet操作**
  - **ルータの起動シーケンス**
  - **コンフィグレーションレジスタ**
  - **Cisco iOSイメージ管理**
  - **コンフィグレーションファイルの管理**
  - **Cisco iOSイメージのライセンス**
  - **SNMP**
  - **Syslog**
  - **NetFlow**


- コンソールパスワード設定
```
(config)#line console 0
(config)#password <password>
(config)#login
 -> loginコマンドによってログイン時の認証機能が有効になる
```
- パスワード反映するために再起動は不要
- コンソールポートのパスワードはデフォルトでは暗号化されない


- vty(仮想端末)
  - 仮想ポートなので、複数設定が可能
  - ライン番号は0〜指定が可能
  - 複数設定すると同時に複数の管理アクセスが許可される
  - 最大5つまで同時接続が可能
  - vtyポートのデフォルトはloginコマンドあり
  - `no login`にすると、パスワードなしでリモート接続が可能
    - パスワード認証が無効化される 
[パスワードの設定]
```
(config)#line vty <line-number>
(config)#password <password>
(config-line)#login
```
- Ciscoルータ
  - デフォルトでは5個(0-4)のvtyポートに対してloginコマンドが設定され、パスワードなし
    - vtyパスワードを設定する必要がある(telnetするには) 

- **特権モードパスワード**
  - ユーザEXECからenableコマンドを実行して特権EXECモードに移る時に要求されるパスワード 

[特権モードパスワードの設定, 2種類]
```
- イネーブルパスワードの設定(暗号化なし)
(config)#enable password <password>

- イネーブルシークレットパスワードの設定(暗号化あり MD5)
(config)#enable secret <password>


-> show running-configした場合に、暗号化なしだと平文で、暗号化有りだと暗号化されて表示される
```

- show running-configで平文のパスワードを暗号化する方法
  - コンフィグレーションモードで
    - `(config)#service password-encryption` 

- VTY(仮想端末)へ管理接続を許可する
  - sshだけ 
    - `transport input ssh`コマンド 
  - telnetとssh
    - `transport input telnet ssh`
  - すべて許可する
    - `transport input all` 

- **ローカル認証**
  - ユーザアカウントをコンフィグレーションファイル内に格納して、ルータ自身で認証を行う
[ユーザか買うんとの作成]
```
(config)#username <username> password <password>
 -> パスワードは平文
or 

(config)#username <uesrname> secret <password> 
 -> パスワードは暗号化
```

- Ciscoルータにリモートでssh接続する場合
  - ユーザアカウントの作成
  - ホスト名の設定
  - ドメイン名の設定
  - 暗号鍵の生成
  - sshバージョンの設定
  - sshの許可
  - ローカル認証の有効化
[ssh接続のためのコマンド]
```
- ユーザアカウントの作成
(config)#username <username> password <password>
or
(config)#username <username> secret <password>

- ホスト名の設定
(config)#hostname <hostname>

- ドメイン名の設定
(config)#ip domain-name <domain-name>

- 暗号鍵の生成
(config)#crypto key generate rsa

- SSHバージョンの指定
(config)#ip ssh version {1 | 2}

- SSHのみ許可
(config)#line vty <line-number>
(config-line)#transport input ssh

- ローカル認証の有効化
(config-line)#login local
```

- **exec-timeoutコマンド**
  - 一定時間何も操作してないと、iOSは自動的にセッションを切断する
    - セキュリティ的なところが理由
    - コマンド
      - `(config-line)#exec-timeout <minutes> [<seconds>]` 
    - デフォルトではタイムアウトは10分
    - `exec-timeout 0`にするとタイムアウトしなくなる

- **バナーメッセージ**
  - ログイン時にバナーメッセージを表示する
  - **motdバナー(Message of The Day(本日のメッセージ))** 
  - 管理者間で共有するときなどに使われる
[バナーメッセージの設定]
```
(config)#banner motd <delimiter> <メッセージ内容><delimiter>

*) <delimiter>でメッセージの最初と最後を区切る
#で区切る
```

- CDP(Cisco Discovery Protocol)
  - 隣接するCiscoデバイスを検出し、プロトコルやアドレス情報などを知ることができるCisco独自のプロトコル 
  - CDPを活用すると、間違ったケーブリングやアドレス設定などのトラブルシューティングを行うことができる
  - データリンク層で動作するプロトコル
  - CDPで隣接デバイスのIPアドレスがわかる
  - CDPはデフォルトで有効になっている
  - 60秒ごとにマルチキャストで送信される
  - SNAPをサポートする物理メディアで利用出来る
  - CDPメッセージを受信しても転送することはないので、Ciscoデバイスと直接接続している必要がある
  - `show cdp neighbors`
    - CDPで検出された隣接されたデバイスの要約情報を表示する
  - `show cdp entry *`コマンド
    - CDPで検出された全ての隣接デバイスの詳細情報を表示する 

- Telnet
  - セッション中断: `Ctrl + Shift + 6 => x` 
  - `show sessions`: Telnet接続を確認する方法
  - セッション中断状態で `Enter`を押すと、中断している直前のTelnetセッションが再開される
  - 中断中のセッションが複数ある場合、`resume + [セッションのコネクション番号]`コマンドを使えば再開することができる
  - `disconnect + [セッションのコネクション番号]`コマンドを使えば、切断できる

- デバッグで、プロセスに関する動作をリアルタイムに確認
  - `terminal monitor`コマンド 
  - タイムスタンプの設定
    - `(config)#service timestamp debug [ uptime | datetime | msec]` 

- アクティブなプロセスに関する情報を表示する
  - show processes 

- `show startup-config`
  - NVRAMに保存された設定で、次回の起動時に適用される

**[Ciscoのメモリ]**

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/cisco-mem-fig.png)


|種類 　　   |説明         |
|:-----------|:------------|
|ROM(Read Ony Memory)| 読み込み専用のメモリで、ルータを起動するためのマイクロコードやパスワードリカバリなど、障害復旧を行うためのプログラムが格納されている |
|RAM(Random Access Memory)|読み書き可能. 電源が切れると切ろうした内容が消えてしまう. 稼働中のコンフィグファイルであるrunning-config,iOS、ルーティングテーブル, ARPテーブルなどが保存される|
|NVRAM (Non-Volatile Random Access Memory)|電源を切っても消えない.不揮発性ランダムアクセスメモリstartup-configなどを格納.コンフィグレーションレジスタも格納|
|FLASH|自由に読み書き可能なメモリ, ルータの電源を切っても内容は消えない. iOSソフトウェアのイメージ、ルータの起動時にブートストラップによってRAM上に展開して読み込まれる|


**[Ciscoルータの電源を入れたときに実行されるイベント順序]**

1.  **POST(Power-On Self-Test)の実行**
  - コンポーネント(CPU, IF、メモリ)が機能するか確認
2.  **ブートストラップコードのロードと実行**
  - Cisco iSOソフトウェアの検出、RAMへのロード, Cisco iOSソフトウェアの実行などのイベント実行
  - ブートストラップは実行するCisco iOSソフトウェアを特定するためにNVRAM内のコンフィグレーションレジスタのブートフィールド値をチェックする
3.  **Cisco iOSソフトウェアの検索**
  - ブートストラップにより、実行するCisco iOSソフトウェアが決定される
4.  **Cisco iOSソフトウェアのロード**
  - 適切なCisco iOSソフトウェアイメージを検出すると、それをRAMに読み込んでロードする
  - ほとんどの機種でiOSは圧縮されており、RAM上に展開して起動する
5.  **コンフィグレーションファイルの検出とロード**
  - NVRAMに保存済みのコンフィグレーションファイル(startup-config)が存在する場合、それをRAMに読み込んでrunning-configとして実行する。
  - NVRAMにstartup-configが存在しない場合は、セットアップモードによって設定を開始するかを問うメッセージを表示する
6.  **Cisco iOSソフトウェアの実行**
  - running-configを使用してCisco iOSソフトウェアを実行してプロンプトが表示される  


**コンンフィグレーションレジスタ**
- パスワードリカバリ
- ROMモニタで強制的にブートする
- ブート元及び、デフォルトのブートファイル名を選択する
- ブロードキャストアドレスを制御する
- コンソールの回線速度を変更
- **show versionコマンド**で内容を確認できる
- レジスタ値は**『0x2102』**(16bit値)
- 下位4bit(bit番号0〜3)を**ブートフィールド**という

- **boot systemコマンド**
  - Cisco iOSソフトウェアイメージの場所とファイル名を指定することで管理者はiOSの起動を制御できる - `(config)#boot system { flash <filename> | tftp <filename> <server-address> | rom }`
    - iOSのイメージソースは**フラッシュメモリ**, **TFTPサーバ**, **ROM** 
    - 通常はフラッシュメモリからロードするが、どこからロードしたかは、**show version**で確認可能

- 新しいiOSにシステムをアップグレードする場合、入手した新規シメージをTFTPサーバやFTPサーバからダウンロードする必要がある。

- TFTPサーバからダウンロードする場合は
  - **特権EXECモード**で `copy tftp: flash`コマンド 
- フラッシュメモリのファイルをTFTPサーバへコピー
  - `#copy flash: tftp`
- ルータの現在のコンフィグレーション(running-config)をTFTPサーバへコピーする場合
  - `copy running-config tftp:` 
 
  
- **ポートセキュリティ**
  - Layer2のセキュリティ機能
  - あらかじめ許可しているMACアドレスからのフレームを転送し、それ以外のMACアドレスからのフレームをブロックする
  - 許可されたMACアドレスを**セキュアMACアドレス**と言う
    - パターンは3通り
  - 許可されていない送信元MACアドレスのフレームを違反フレームという
  - 許可した最大数を超えて、アドレステーブルに未登録の送信元MACアドレスを持つフレームを受信したらセキュリティ違反が発生したと判断
    - 違反モード3種類(デフォルトはshutdown)
      - `protect`
      - `restrict`
      - `shutdown`
        - 検出するとポートを即座にerr-disabled状態にして、SNMPトラップを送信して違反カウンタを増やす 

|種類        |説明         |
|:-----------|:------------|
|スタティック|手動で登録し, MACアドレステーブルとrunning-configに保存される|
|ダイナミック|フレームを受信することで動的に送信MACアドレスがMACアドレステーブルにのみ保存され、スイッチの再起動時に消去される|
|スティッキー|フレームを受信することで動的に送信元MACアドレスがMACアドレステーブルとrunning-configに保存される|

- **スティッキーラーニング**
  - スティッキーラーニングによってセキュアMACアドレスを動的に学習し、running-configに保存できる

**[ポートセキュリティの設定手順]**

1.  アクセスポートに設定
2.  ポートセキュリティの有効化
3.  許可するMACアドレスの最大数の設定
4.  許可するMACアドレスの設定
5.  違反時のアクションの設定

(※) デフォルトでは許可されるMACアドレスの最大数は『1』
-> 1にしておくと、リピータハブを接続されるのを防ぐことができる

**[設定コマンド(例)]**
```
(config)#interface fastethernet 0/1
(config-if)#switchport mode access
(config-if)#switchport port-security
(config-if)#switchport port-security mac-address stikcy

(設定の確認)
#show port-security
```

[ntpサーバとつなぐ]
```
グローバルコンフィグレーションモードで
(NTPクライアントとしての設定)
(config)#ntp server <ip-address>

(自身をNTPサーバとする設定)
(config)#ntp master
```


- **UDI**(Unique Device Identifire)
  - 固有デバイス識別情報
    - PIDはCisco IOSイメージのライセンスを取得する際に必要なUDIに含まれる製品番号 
    - PID(Product ID)とSN(Serial Number)の2つから構成される
    - `show license udi`で確認できる



![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/snmp-image2.jpg)



- **SNMP**
  - **SNMPマネージャ**
    - エージェントから必要な情報を収集して、ネットワーク機器を管理するデバイス
    - 収集した情報を解析し、その結果をレポートやグラフに加工できる
  - **SNMPエージェント**
    - 管理対象機器(ルータ,スイッチ,サーバなど)の情報収集を行う機器または常駐するプログラム
  - **MIB**
    - 管理対象オブジェクト(情報)で構成される管理データベース 
