### Chapter3 Cisco IOSソフトウェアの基礎

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

- Pingコマンド(@Cisco iOS)
  - 定義した時間内にエコー要求パケットが宛先に到着し、エコー応答を受信した場合にのみ成功と判断
  - User EXECモードまたは特権EXECモードでpingが使用できる
  - `ping <target_ip>`を`p <target_ip>`と省略できる
  - デフォルトではICMPエコー要求パケットを5つ送信し、タイムアウト時間は2秒になっている
  - ping実行時の表示記号
    - `!` : ICMPエコー応答を受信
    - `.` : ICMPエコー応答を受信できずにタイムアウトした
    - `U` : ICMP宛先到達不能(Destination Unreachable)を受信
  
