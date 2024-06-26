# ESP32でUDP通信を始める
## UDP通信とは
通称、User Datagram Protocolで動画の再生など、時間を重視する際によく用いられる通信です。
また、通信するデバイスが同じWiFi上にある必要があります。

## 下準備
ESP32で通信するためのライブラリ（他の人が実装してくれたコードをまとめたもの）があるのですが、これを使うために以下のようにしてください。

```cpp
// #include <使いたいライブラリ名>みたいな感じですね。
#include <WiFi.h>
#include <WiFiUdp.h>
```

## ESP32をアクセスポイントにする
先ほど同じWiFi上にしなければいけないと書きましたが、マイコンとPC間で通信するのですが、なんとESP32というマイコンがWiFiのアクセスポイントになれるらしいです。なので今回はESP32をWiFiの発信源にしていきます。なのでまず下のようにWiFiの名前とパスワードの変数を作成します。「TestServer」という名前のWiFiを「test1000」というパスワードで実装します。

また、このような変数はどの関数の中でもない**#include**の下とかに書きます。（グローバル変数にするため）

`※グローバル変数：通常変数は同じ関数内でしか触ることができないのですが、どこの関数にも入らないように書くと、どの関数でも触れるようになるのです。`
### 変数の設定
```cpp
const char *ssid = "TestServer";
const char *pass = "test1000";
```

そしてUDP通信ではデバイスが特定のIPアドレスとポートをもちます。IPアドレスとはxxx.xxx.xxx.xxxで表される住所のようなものです。そしてポートとはその住所にある建物の部屋番号のようなものだと私は考えています。

```cpp
const IPAddress ip(192, 168, 4, 1);
const IPAddress gateway(192, 168, 4, 1);
const IPAddress subnet(255, 255, 255, 0);
```

新しい型が出てきました。先ほど**include**したライブラリのおかげで**IPAddress**という型を使用することができます。カンマ区切りの4つの数字を関数の引数のように入れることで設定することができるんですね。いい忘れてましたが、すべての変数の前についてる**const**というのはconstantの略で、これをつけた変数はこれから中身を変えることができません。絶対に変わらないし変わってほしくない時につけるといいと思います。

### WiFiサーバーを実装する
以下はsetup関数内に入れてください。

```cpp
void setup()
{
    WiFi.softAP(ssid, pass);
    WiFi.softAPConfig(ip, gateway, subnet);
}
```

こちらも先程同様、**include**したライブラリにより提供されている関数です。**WiFi.h**を**include**することによりWiFiというクラスを使うことができます。

`※クラスについては長くなりそうなので省略させていただきます。複数の変数、関数を格納できる型みたいに思ってください。`

これで先程宣言した変数を反映することができました。これによりESP32をWiFiのアクセスポイントにすることができました。（もちろんネットの動画再生とかは無理ですよ）
ちなみにこのような無線通信には帯域があり、ESP32で出せるのは**2.4GHz**です。

## UDP通信をする
### セットアップ
先程説明したポートを以下のように定数（constant）に宣言してください。（0~65535の数字）
```cpp
const uint16_t port = 10000;
```
また、UDP通信するためには**WiFiUDP**という型で変数を宣言する必要があります。
```cpp
WiFiUDP udp;
```
これらの変数を実行することができたら、setup関数内で以下を実装します。

`※WiFiUDPによる通信は先程のWiFiがあって動くものです。`

```cpp
void setup()
{
    WiFi.softAP(ssid, pass);
    WiFi.softAPConfig(ip, gateway, subnet);
    
    udp.begin(port);
}
```

### UDP通信によりメッセージを送信する
いよいよメッセージを送信する部分についての解説です。そもそもUDP通信ではパケットというものを作成して送信しています。パケットとは通信するために情報を細切れにしたようなものです。
これには通信相手の情報も必要なので仮に以下のようにします。
```cpp
const IPAddress remote_ip(192, 168, 4, 2);
const uint16_t remote_port = 8080;
```

それでは実際に送信するコードを記述します。こちらはloop関数内です。

```cpp
void loop()
{
    udp.beginPacket(remote_ip, remote_port);
    udp.print("Hello ESP32");
    udp.endPacket();
}
```

パケットの始まりと終わりをつけることによって情報を送信することができました。**beginPacket**と**endPacket**の間に送りたい文章を入れただけです。

### メッセージを受信する
どちらかというとロボットを動かすマイコンでは受信のほうが使うことが多い気がしますね。
通信によりメッセージを受信する場合にはバッファというものをつくります。これは受信内容を保存する先のようなものだと思います。まずはこれをグローバル変数の文字型の配列として用意します。

```cpp
char buf[256]
```

そして以下のようにloop関数内に実装するだけです。

```cpp
void loop()
{
    udp.read(buf, 256);
    Serial.println(buf);
}
```

情報の受信には相手の情報（IPアドレスやポート）はいりません。
**read**関数では保存先のバッファと読み取る文字数を指定します。
下の**Serial.println**で受信した内容を表示しています。


## おわりに
まとめました。
```cpp
#include <WiFi.h>
#include <WiFiUdp.h>

const char* ssid = "TestServer";
const char* pass = "test1000";
const uint16_t port = 10000;

const IPAddress ip(192, 168, 4, 1);
const IPAddress gateway(192, 168, 4, 1);
const IPAddress subnet(255, 255, 255, 0);

const IPAddress remote_ip(192, 168, 4, 2);
const uint16_t remote_port = 8080;

WiFiUDP udp;

char buf[256];

void setup() {
  Serial.begin(115200);
  Serial.printf("Start Connecting");

  WiFi.softAP(ssid, pass);
  WiFi.softAPConfig(ip, gateway, subnet);

  Serial.println("\nWiFi is Connected");

  udp.begin(port);
  Serial.println("Start UDP");
}

void loop() {
    udp.read(buf, 256);
    Serial.println(buf);

    udp.beginPacket(remote_ip, remote_port);
    udp.printf("Hello, ESP32");
    udp.endPacket();
}
```

[チュートリアル一覧に戻る](../index.md)