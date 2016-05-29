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
