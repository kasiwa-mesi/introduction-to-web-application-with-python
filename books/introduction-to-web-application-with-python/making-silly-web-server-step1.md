---
title: "STEP1: ChromeとApacheで通信する"
---

# ChromeとApacheで通信する

それではまずは自分のマシンの中で、ブラウザである**Chrome**と、Webサーバーである**Apache**を起動し、互いに通信させてみることにしましょう。
下図の通信ができるようになっていることが目標です。

![](https://storage.googleapis.com/zenn-user-upload/gpz813pg7ni0ru16tj4det75xtf4)

# STEP1-1: Chromeをインストールして起動する
本書を読まれている皆さんであれば、ブラウザをインストールするのにアレコレと説明は不要でしょう。
既にインストール済み方はそのまま使っていただいて結構ですし、まだインストールされていない方は公式サイトよりダウンロードして起動してください。

公式サイト：https://www.google.com/intl/ja_jp/chrome/

このような画面が開いていれば起動完了です。

![](https://storage.googleapis.com/zenn-user-upload/0syrmyaip7tu9qg6ji0800wvt7dh)

# STEP1-2: Apacheをインストールして起動する
## インストール
MacOSにはApacheが標準でインストールされていますので、特別にインストールをする必要はありません^[Homebrewなどでパッケージのバージョン管理を集約したい方は、別途`brew install httpd`などでインストールしていただいても構いません。]。

念の為、ターミナル^[Macに標準でついている`ターミナル`というアプリケーションで構いません。IDEのターミナルやiTermなどのお気に入りのターミナルがある方は、そちらを使っていただいても構いません。]を開いていただいて`apachectl -v`とコマンドを実行してバインストールが完了していることを確認してみましょう。
バージョン情報が表示されれば、Apacheはインストールされています。

ちなみに、`apachectl`はApacheを起動したり終了させたりする為のプログラムで、apache-controlから来ています。

```shell
$ apachectl -v
Server version: Apache/2.4.41 (Unix)
Server built:   Jun  5 2020 23:42:06
```

## 起動
次にApacheを起動させてみます。
起動させるコマンドは、`apachectl start`です。

ですが、apacheの起動時にはマシンの特別な権限が必要なため、rootユーザーで起動してあげる必要があります。
rootユーザーとしてコマンドを実行する`sudo`と組み合わせて、`sudo apachectl start`と実行してあげましょう。
初回はMacユーザーのパスワードを聞かれますので、入力してください。
```shell
$ sudo apachectl start
Password:
```

Apacheは少し不親切なところがあって、起動に成功しても何もメッセージを表示してくれません。
不安になりますが、逆にエラーメッセージが何も表示されなければきっと起動に成功しています。

## 起動確認
念の為、Apacheが起動していることを確認しておきましょう。

rootユーザーが実行しているプログラムの中から`httpd`というプログラムを抜き出してくるコマンド`ps -u root | grep httpd`を実行してみましょう。
ちなみに、httpdというのはApacheのことです。製品名とプログラム名が違ってややこしいですが、そういうものです。

下記のように、何か1行でも表示されればApacheは起動しています。
```shell
$ ps -u root | grep httpd
    0 47333 ??         0:00.25 /usr/sbin/httpd -D FOREGROUND
```

# STEP1-3: ChromeとApacheで通信してみる
さて、ここまでの作業（=ChromeとApacheの起動）で、あなたのマシンは下図のような状態になっています。

![](https://storage.googleapis.com/zenn-user-upload/4xvuwp7thio7zcnkbpoh8bnxj3e8)

Chrome（ブラウザ）というプログラムは起動はしていますが、Webサービスを利用する側なのでユーザーから司令を与えられるまで何もせずじっとしています。

また、Apache（Webサーバー）というプログラムも起動しており、こちらはWebサービスを提供する側になりますが、いつユーザーからリクエストがくるか分からないのでずっと通信の窓口（ポート80）を見張っている状態です。

※　ポートについては、この後説明します。

さて、これからChromeとApacheで通信を行うのですが、この作業は結論から言ってしまうと

「ChromeのURLバーに`http://localhost/`と入力してみてください」

で終わりになります。
また、ほとんどの入門書には実際そのように書かれています。

しかし、この作業が意味するところをしっかりと理解しておくことが今後の開発を理解する助けになるため、少し冗長に説明をしておこうと思います。

## ブラウザとWebサーバーが「通信をする」とは
これまでChromeとApacheで**通信をする**と言ってきましたが、これは具体的には

**ChromeからApacheへ向かって(Webサービスを提供してほしいという)リクエストを送り、Apacheがそれに対してレスポンス（Webサービス）を返す**

ということを意味しています。

とにかく、最初にやらなければいけないことは、ChromeからApacheへ「リクエストを送る」ということです。

:::details 「Webサービスを返す」
「Webサービスを返すってなんだ？」と疑問に思う方もいらっしゃるかもしれませんが、それは非常に正しい疑問です。
Webサービスは抽象的な言葉で、手にとって「どうぞ」と渡せるような具体的なものではないからです。

これを理解するには、私達が「Webサービス」と読んでいるものの実態（具体的なモノ）が何かを知る必要があります。

しかし、それは後々明らかになっていくことですので、今は置いておいてください。
:::


## ブラウザからWebサーバーへリクエストを送るためには？

あるプログラムがインターネットを通じて別のプログラムへ何かを送るとき、必要になるのは「宛先」と「内容」です。

この2つのうち、Chromeでは **「宛先」だけURLバーに入力してあげればOK** です。
「内容」は自動的に生成して送ってくれます。

また、Webサービスにおける宛先のことを、URLと呼び、以下のような形式で表現されます。

**URL =** `<protocol>`**://**`<host>`**:**`<port>`**/**`<path>`**?**`<query>`

`protocol`: リクエストの送り方（プロトコル）
`host`: インターネット上の送り先の住所
`port`: 相手のマシン内において、Webサーバーが見張っているポート番号
`path`: 欲しい情報が置いてある場所
`query`: 追加の情報

例）
- `https://zenn.dev/bigen1925/books/introduction-to-web-application-with-python`
- `http://132.45.33.111:2300/foo/bar?name=bigen1925`
- `http://localhost/`

ここの説明がなかなか難しいのですが、よく郵便に例えて説明されることが多く、僕もそれに倣って例えてみます。

### protocol
`protocol`は、送る内容や宛先には関係なく、「送り方」を指定します。

例えば、郵便の送り方として、「普通郵便」の他に「本人限定受取」というものがあります。
普通郵便であれば、送り主は郵便ポストに投函するだけですし、届いた時も受け取り主は不在でも配達員さんがポストに入れておいてくれます。
しかし、「本人限定受取」の送り方をする場合は、送り主は場合によっては事前申し込みをした上で郵便窓口に行き、受け取り主も配達員さんに写真付き身分証明書を見せないといけません。

このように、宛先や内容とは関係ないとはいえ、「送る手順」「受け取る手順」が異なる送り方をする場合はそれを明示してあげないと受け取り手も困ってしまう訳です。

Webサービスの世界では、`http`や`https`というprotocolがよく使われています。
httpはいわば普通郵便で、送りたい内容をそのまま送りつけます。
httpsは暗号化通信で、送り主は暗号化してから送り、受け取り手は受け取ったあと復号してから内容を読みます。

`https://~`とブラウザにURLバーに書いた場合には、ブラウザに「暗号化してから送ってね〜」と伝えることになります。

::: message
httpsは本当はもっと複雑なことをしていますが、ここでは説明を割愛させてください。
:::

暗号化してWebサーバーに送った場合は、当然Webサーバーはその内容を復号する機能を実装していなければ、内容を見ることができません。
本書ではそこまで複雑な機能を作り込むつもりはありませんので、httpしか出てこないと思っていてください。

今回のChromeとApacheの通信においても、httpプロトコルを用いるため、
`http://~~`
というURLにアクセスすることになります。

### host
`host`は、送り先のWebサーバープログラムが動いているマシン（= コンピュータ)のインターネット上の住所を示します。

郵便でも、送り先の建物の住所を書きますよね、これがhostにあたります。
例えば、`東京都hoge区fuga町10-24`とかです。

基本的にはインターネット上であるマシンを特定するためには、IPアドレスを使います。
（IPアドレスについての説明はここでは割愛します。）

しかし、IPアドレス以外にも`host`として使えるものとして、

- DNSに登録されたドメイン。　`zenn.dev` `google.com`など。
- Macであれば`/etc/hosts`に記載されたエイリアス。 `my-server.piyo 123.45.6.78`と書いておくと、`my-server.piyo`はIPアドレス`123.45.6.78`と同じとみなされる。
- localhost。　これは世界的にIPアドレス`127.0.0.1`とみなされ、自分のPCを指す。

などがあります。

今回のChromeとApacheの通信においては、ChromeとApacheは同じマシン内で起動していますので、
`http://localhost:~~`
を使うことになります。


ところで気をつけなければいけないのは、建物の住所があれば送り先は特定できそうなものですが、この建物がマンションだった場合、部屋番号も併せて書かなければ目的の送り主には郵便が届かないということです。

プログラムも同様で、1つのマンションの中にも様々な家族が住んでいるように、1つのマシンの中には通常たくさんのプログラムが動いています。
例えば僕のPCでもWebサーバー、Discord、Slack、Kindleなどが起動しています。
ブラウザがリクエストを送る際には、ピンポイントにWebサーバープログラムに送りつける必要があるため、建物住所にあたる`host`だけでは情報が不足しています。

そこで追加で必要になるのが`port`です。

### port
`port`は、インターネット通信の際に特定のマシンの中で動いている複数のプログラムから目的のプログラムを特定するための番号です。

郵便では部屋番号にあたります。

ブラウザは、僕のPCの中で動いているDiscordではなく、Slackでもなく、Webサーバーへリクエストを送りたいのです。
そのため、Webサーバーに割り振られているport番号へリクエストを送信する必要があります。

port番号は、プログラム起動時にプログラムが 0番 ~ 65535番 の中から自分で設定することができます。
Apacheの起動時には説明を先送りにしましたが、**Apacheは起動時に自分にポート番号80番を割り振っています。**

そのため、今回のChromeとApacheの通信においては、80番ポートを使って
`http://localhost:80/~~`
へアクセスする必要があります。

ただし、Chrome（とその他多くのブラウザ）では、
「httpプロトコルで通信するときはデフォルトで80番ポートへ向けて通信する」と決まっているため、
httpでポート番号が80番の場合に限り、portを省略することができるようになっています。

例えば、
`http://localhost:80/~`
と書くのと
`http://localhost/~`
と書くのは同じ意味になります。

本書では、今後後者の記法で進めていきます。

::: details コラム：well-known ports
port番号はプログラムが自分で割り振ることが可能ですが、複数のプログラムで同じport番号を使うことはできず、後からそのport番号を割り振ろうとした方がエラーとなってしまいます。

そのため、どのマシンでもよく使うようなプログラムには予めport番号が予約されており、例えば80番はHTTPプロトコルの通信のために置いておきましょうということになっています。

予約といっても、HTTP通信じゃないプログラムが80番を使ったからといって、ただちにエラーになるようなものではありません。
みんなの共通認識というか、マナーというか、そういうレベルの約束事です。
しかし、世界的に標準化されたマナーの1つなので、他のマシンは「80番といえばHTTP通信でしょ！」といって無理矢理通信してくるかもしれませんし、守っておいたほうが良いでしょう。

こういった予約されたport番号は[well-known ports](https://ja.wikipedia.org/wiki/TCP%E3%82%84UDP%E3%81%AB%E3%81%8A%E3%81%91%E3%82%8B%E3%83%9D%E3%83%BC%E3%83%88%E7%95%AA%E5%8F%B7%E3%81%AE%E4%B8%80%E8%A6%A7#%E3%82%A6%E3%82%A7%E3%83%AB%E3%83%8E%E3%82%A6%E3%83%B3%E3%83%9D%E3%83%BC%E3%83%88%E7%95%AA%E5%8F%B7_(0%E2%80%931023))として知られており、 0番 ~ 1023番 のポートはなにかしらの予約がされています。

Apacheが起動時に80番ポートを割り振っていると説明しましたが、これはApacheはHTTP通信を行うWebサーバーなので、起動時のポート番号のデフォルト値が80番に設定されているためです。
ただし、セキュリティ上の都合などであえて80番ではない番号を割り振りたい場合は別途設定することができます。

またChromeのhttp通信のデフォルトが80番なのも、このwell known portで予約されており、ブラウザを使った通信でわざわざ80番以外を指定するケースがほぼないからです。

ちなみに、https通信は443番ポートが予約されており、
`https://zenn.dev:443/~`
と書くのと、
`https://zenn.dev/~`
と書くのは同じ意味になります。
:::

### path
`protocol`, `host`, `port`が揃えば、ブラウザは相手のWebサーバープログラムの場所を特定して、通信を始めることができます。

しかし、一般的には1つのWebサービスに対する要求は1種類ではありません。
例えば、「記事の一覧を見せて欲しい」というときもあれば、「ユーザーの詳細情報を見せて欲しい」というときもあれば、「ユーザー登録がしたい」ときもあれば、「オンラインブックの第5章を書きたい」というときもあるでしょう。

そのWebサーバーに対して、どのようなサービスを要求したいかを伝え分けるために、pathにその情報を追加します。

例えば、`Zenn`というサービスに対して、`bigen1925という人が書いた本の、伸び悩んでいる（以下略）というやつを読みたい`ということを伝えるには、pathに`/bigen1925/books/e6c9492a82f5e2e10fca`と書きます。

今回のChromeとApacheの通信においては、pathには`/`を使います。
Apacheは本来ただのWebサーバーなので、ユーザーに提供したい色々なページは私達が追加で用意してあげる必要があります。
しかし、デフォルトでも動作確認用に`/`というpathには`It works!`と表示してくれる機能が用意されていますので、そちらを要求してみましょう。

ですので、今回使うURLはここまでと合わせて
`http://localhost/?~`
ということになります。

### query
`query`は、pathに加えて何か情報を追加で送りたいときに使います。
送る情報（パラメータ）は、名前と値を`=`で区切り、パラメータ同士は`&`
たとえば、
`https://www.google.com/search?q=google&sourceid=chrome`
というURLは、
「`/search`（Google検索）という種類のサービスを受けたくて、検索ワードは`google`で、chromeから検索してきたよ」
ということを伝えることになります。

このqueryはOptional(=省略可能)で、サービスごとに必要な場合のみ付与すれば良いことになっていますので、今回は省略してしまいます。

結果、今回の実験では、
**「ChromeとApacheで通信を行うには、ChromeのURLバーに`http://localhost/`と入力すればよい」**
ということになります。

::: details コラム: pathとquery
鋭い読者は、「pathとqueryって分かれてる意味あるの？」と思ったかもしれません。

例えば、pathの例としてあげた
`/bigen1925/books/e6c9492a82f5e2e10fca`
は、
`/bigen1925?book=e6c9492a82f5e2e10fca`
ではだめなのか？　ということです。

結論からいうと、Webサービス側がそのようなパラメータの受け取りに対応していれば、どちらでも問題ありません。

何故分かれているかを理解するには、少し歴史をたどる必要があります。

HTTPやURLという技術が生まれた当時は、Webでは静的ファイルのやりとりが主用途として想定されていました。
そのため、pathとファイルは1:1で対応しており、pathが同じであれば返ってくるWebページは（ファイルが更新されなければ）いつも同じでした。
queryは「googleから検索してきたよ」とか、「トップページの右上にあるボタンを押してやってきたよ」とか、ページの表示内容は同じでいいんだけど付加情報を伝えたい、という時に使われるものでした。

ところが、Web技術の発達に伴いpathとファイルが1:1ではなくなり、同じpathでも毎回違った結果を返すことができるようになりました。
そうしてしばらくすると
「`/search`を見たいっていっても、色々な形式があるんだよね。具体的に何を見たいかqueryを使って教えてくれる？」
といった方式のWebサービスが増えてきました。

その結果、従来の「レスポンスを変えたいならpathを変える。レスポンスを変えずに何か伝えたいならqueryを使う。」という用途は少しずつ崩れてしまい、pathとqueryの棲み分けが怪しくなってしまったというわけです。

今でもページの内容が変わるならパスにパラメータを含め、ページの内容が変わらないならクエリにパラメータを含めるようにするのがお行儀が良いとされていますが、個人的にはもはや拘る必要はないと思っています。
:::

## ChromeからApacheへリクエストを送る
さて、随分長くなってしまいましたが、実際にChromeのURLバーに`http://localhost/`と打ち込んでEnterを押してみましょう。

プログラム的には下図をしていることになります。

![](https://storage.googleapis.com/zenn-user-upload/7ifzdeu5ep46jgatktt3zty2owhm)

ブラウザに、このような質素な画面が表示されれば成功です。

![](https://storage.googleapis.com/zenn-user-upload/1jsmt46i35o1l81ebj24vh55ux05)

この質素な`It works!`という画面が、ApacheというWebサービスが提供してくれたサービス、というわけです。
いやー長かった。

## Apacheを終了させる
Apacheは一回起動すると、自分で終了させるかPCをシャットダウンするまで起動しっぱなしになってしまいます。
なので、実験が終わったら、Apacheを終了させておきましょう。

Apacheを終了させるには、ターミナルで`sudo apachectl stop`を実行します。

```shell
$ sudo apachectl stop
/System/Library/LaunchDaemons/org.apache.httpd.plist: Operation now in progress
```

「オペレーション（=終了）は今実行中ですよ」というメッセージが表示されていますが、Apacheはすぐに終了します。
念の為、apacheが本当に終了しているか確認してみましょう。以前出てきた`ps -u root | grep httpd`をターミナルで実行します
```shell
$ ps -u root | grep httpd
$
```
コマンド実行のあとに何も表示されないということは、Apacheが起動していない（＝終了できている）ということになります。

# 次回
ここに辿り着くまでたくさんの説明をしてきましたので、さすがに読むのが疲れてきたと思います。
次章からはPythonで実際にプログラムを書くフェーズに入っていきますので、乞うご期待。
