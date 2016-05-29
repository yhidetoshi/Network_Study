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
[パスワードの設定]
```
(config)#line vty <line-number>
(config)#password <password>
(config-line)#login
```

