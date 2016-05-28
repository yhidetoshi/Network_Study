#### Chapter12 インターネット接続
- **内容**
  - **DHCPによるインターネット接続**
  - **NATおよびPATの概要**
  - **NATの設定**
  - **PATの設定**
  - **NATおよびPATの検証**
  - **NATおよびPATのトラブルシューティング**

- **DHCP**
  - 機能
    - IPアドレスなどの設定情報をクライアントに自動割り当て
    - DHCPサーバでプールされたアドレス範囲から、IPアドレスの割り当てと更新を行う
  - 設定可能な情報 
    - IPアドレス
    - サブネットマスク
    - リースの有効期限
    - デフォルトゲートウェイのIPアドレス
    - ドメイン名
    - DNSサーバのIPアドレス
    - NTPサーバのIPアドレス

[DHCPサーバとクライアント]
  

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/dhcp-ser-cli.jpg)

- DHCPクライアントはDISCOVER(ブロードキャスト)メッセージを送信してDHCPサーバを発見する
- DHCPサーバはREQUESTメッセージ(ブロードキャスト)をクライアントに送信する
- DHCPサーバはアドレス競合を検出するためにpingを使用し、DHCPクライアントはGratuitous ARPを使用
- アドレス競合を検出すると、そのアドレスはプールから取り除かれ、管理者が競合を解決するまで割り当てられない
- Ciscoルータでは`ip address dhcp`コマンドを設定

- Cisco iOSはDHCPサーバとしての機能を全て備えている.Ciscoルータ、CatalystスイッチをDHCPサーバとして動作させることが可能
```
- DHCPプールの作成
(config)#ip dhcp pool <pool-name>

- ネットワークアドレスの指定
(dhcp-config)#network <network> [ <subnet-mask> | /<prefix-length> ]

- デフォルトゲートウェイの指定
(dhcp-config)#default-router <ip-address>
-> DHCPクライアントに通知するデフォルトゲートウェイを指定する

- 除外するIPアドレスを指定
(config)#ip dhcp excluded-address <ip-address> [ <last-ip-address> ]
```

- DHCPリレーエージェント
  - DHCPクライアントは異なるサブネットにあるDHCPサーバからIPを取得できるようになる
  - `ip helper-address`コマンド 
   
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/dhcp-relay.png)


- DHCPサーバでIPアドレス競合を確認
  - `show ip dhcp conflict`コマンド 

- スタティックNAT
  - 1対1の変換で常に同じアドレスに変換される
  - 外部に対してサーバを公開する目的などで構成される
  - `(config)#ip nat inside source static <local-ip> <global-ip>`

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/static-nat1.png)
(引用：http://goo.gl/r5lnl)
