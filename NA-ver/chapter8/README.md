#### Chapter8 Layer3冗長化
- **内容**
  - **デフォルトゲートウェイの冗長化**
  - **HSRP**
  - **VRRP**
  - **GLBP**

- **FHRP**(First Hop Redundancy Protocol)
  - 複数のLayer3デバイスをグループ化して「仮想ルータ」を構成
  - 仮想ルータのデフォルトゲートウェイをファイルオーバーを実現する冗長化プロトコル
  - Cisco独自なのはしたの二つ
    - **HSRP** 
    - **GLBP**

- **HSRP**(Hot Stanby Routing Protocol)
  - 同一のサブネット上の複数ルータでHSRPグループを形成する
  - 複数の物理的なルータ(実ルータ)をあたかも１台かのように見せる**『仮想ルータ』**
  - グループでは1台をアクティブ、一台をスタンバイを選出する
    - アクティブが落ちるとスタンバイが引き継ぐ
  -　グループメンバ
    - **仮想ルータ**
    - **アクティブ**
    - **スタンバイ**
  - ARPリクエストに対しては『仮想MACアドレスが』返される 
  
**[アクティブルータの選出方法]**
  1.  プライオリティが最大のルータ(defaultは100)
  2.  IPアドレスが最大のルータ(プライオリティが同じ場合)

- アクティブルータの役割
  - ARPリクエストに応答する
  - 宛先MACアドレスが仮想MACアドレスが転送する

**[仮想MACアドレス]**
仮想ルータには、HSRPグループ番号を基にした仮想MACアドレスが自動的に割り当てられる
```
HSRPの仮想MACアドレス = 0000.0c07.ac[HSRPグループ番号(16進数)]
```

- HSRP仮想IPアドレスには、同一サブネット上の未使用のIPアドレスが必要
- アクティブルータでは、仮想IPアドレスと仮想MACアドレスがアクティブである
- インターフェース上で複数のHSRPグループを形成し、ロードバンスが可能
- HSRPグループの範囲は[0-255]
- フレーンテキスト認証とMD5認証をサポートしている

- アクティブルータとスタンバイルータ
  - 互いに定期的にHelloメッセージを送信
    - Holdタイマの時間が経過してもHelloを受信できなければ相手がダウンしたと判断
    - デフォルト値は、`Helloタイマ:3秒`、`Holdタイマー10秒`


**[デフォルトゲートウェイの冗長構成化プロトコル]**
- 種類
  - `HSRP` : Cisco独自、Active/Standy方式
  - `VRRP` : RFC標準、Master/Backup方式
    -　マルチベンダ環境で利用可能 
  - `GLBP` : Cisco独自、Load Balancing方式

[VRRPの図]と[HSRPの図]
(引用：http://bit.ly/1WIskWx)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/vrrp-image.jpg)
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/hsrp-image.jpeg)




