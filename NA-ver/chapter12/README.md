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

