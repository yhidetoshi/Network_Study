# Chapter2 IPv4アドレスとサブネット

※ 設問28を再度確認する

####[IPアドレスとは]

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ipaddress-fig1.png)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ip-classfull.png)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ipaddress-fig2.png)

![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ip-private1.png)
![Alt Text](https://github.com/yhidetoshi/Pictures/raw/master/Network_Study/ip-global-fig1.png)


- ループバックアドレス
  - 127.0.0.1
  - TCP/IPプロトコルスタックが有効であれば常に利用可能
  - 仮想アドレスであり、NICではない

- サブネット化
  - ネットワークサイズを小さくできるため、管理が容易になる
  - ネットワーク上で扱うトラフィックを分割(局所化)することで、全体的なネットワーク量が削減され、パフォーマンスが改善される
  - ネットワークサイズを小さくしてトラフィックを分離することで、ネットワークセキュリティの適用が容易になる
  - 効率的なアドレッシングによって、IPアドレスの枯渇問題を回避できる
 

  
- マイクロセグメンテーション
    - スイッチのポート毎にコリジョンドメインを小さく分割する 
