# Connection Options

**対象読者**: アーキテクト、管理者、アプリケーションおよびスマートコントラクト開発者

コネクションオプションは、コネクションプロファイルと一緒に使われ、ゲートウェイがどのようにネットワークとやりとりするかを*正確に*制御するのに用います。
ゲートウェイを使うことで、アプリケーションは、ネットワークのトポロジではなく、ビジネスロジックに集中することができます。

このトピックでは次の項目について扱います。

* [なぜコネクションオプションが大切なのか](#scenario)
* [アプリケーションがコネクションオプションをどのように使うか](#usage)
* [各コネクションオプションの意味](#options)
* [それぞれのコネクションオプションをいつ使うべきか](#considerations)

## Scenario

コネクションオプションは、ゲートウェイの振る舞いのある一面を指定するものです。
ゲートウェイは[様々な理由](./gateway.html)から重要ですが、その主な理由は、
アプリケーションが、ネットワークの多くのコンポーネントとのやりとりを管理するなかで、ビジネスロジックやスマートコントラクトに集中できるようにしていることです。

![profile.scenario](./develop.diagram.35.png) *図中のそれぞれの点は、コネクションオプションが振る舞いを制御している部分です。これらのオプションは、本文で全て説明されています*

コネクションオプションの一つの例としては、`issue`アプリケーションで使われるゲートウェイが、`papernet`ネットワークにトランザクションを送信する際に`Isabella`のアイデンティティを使うよう指示するものがあるでしょう。
あるいは、ゲートウェイが制御を返すときに、MagnetoCorpの3つの全てのノードがトランザクションをコミットしたことを確認するまで待つというものもあるでしょう。
コネクションオプションによって、アプリケーションは、ゲートウェイがネットワークとやりとりする際の振る舞いを指定することができます。
ゲートウェイがなければ、アプリケーションははるかに多くの処理をする必要があります。
ゲートウェイによって、時間を節約できますし、アプリケーションを読みやすくそしてエラーを減らすことができます。

## Usage

アプリケーションが利用可能なコネクションオプションの[全て](#options)は、この後すぐに説明しますが、まずはサンプルのMagnetoCorpの`issue`アプリケーションでどのように指定されているかを見てみましょう。

```javascript
const userName = 'User1@org1.example.com';
const wallet = new FileSystemWallet('../identity/user/isabella/wallet');

const connectionOptions = {
  identity: userName,
  wallet: wallet,
  eventHandlerOptions: {
    commitTimeout: 100,
    strategy: EventStrategies.MSPID_SCOPE_ANYFORTX
    }
  };

await gateway.connect(connectionProfile, connectionOptions);
```

`identity`と`wallet`オプションが、`connectionOption`オブジェクトの単純なプロパティであることがわかるでしょう。
これらのプロパティはそれぞれ、コードのその前でセットされている`userName`と`wallet`の値をもちます。
それだけで一つのオブジェクトとなっている`eventHandlerOptions`オプションと対比してみてください。
これは、`commitTimeOut: 100` (単位は秒)と`strategy: EventStrategies.MSPID_SCOPE_ANYFORTX`という二つのプロパティをもっています。

`connectionOptions`がゲートウェイに対して、`connectionProfile`に加えて渡されているのがわかります。
コネクションプロファイルでネットワークが識別され、オプションによってゲートウェイがそのネットワークとどのようにやりとりしなければならないかを正確に指定しています。
それでは、利用可能なオプションを見ていきましょう。

## Options

下記が、利用可能なオプションとその意味のリストです。

* `wallet`は、アプリケーションの代わりにゲートウェイが使用するウォレットを識別するものです。図中の**1**を参照してください。
  ここでは、ウォレットはアプリケーションによって指定されますが、実際にアイデンティティを取得するのはゲートウェイです。

  ウォレットは必ず指定する必要があります。
  決めるべき最も重要な点は、ファイルシステムやインメモリやHSMやデータベースといった、使用するウォレットの[種類](./wallet.html#type)です。


* `identity`は、`wallet`の中からアプリケーションが使うユーザーのアイデンティティです。
  図中の**2a**を参照してください。ユーザーのアイデンティティがアプリケーションによって指定され、それはアプリケーションのユーザーであるIsabella(**2b**)を代表するものです。
  アイデンティティは、実際にはゲートウェイが取得します。

  この例では、Isabellaのアイデンティティは、異なるMSP(**2c**と**2d**)によって、彼女がMagnetoCorpの者であり、ある役割を持っていることを識別するのに使われます。
  この二つの事実は、それに応じてリソースに対する彼女の権限、たとえば台帳を読んだり書いたりすることができるかどうかを決定します。

  ユーザーのアイデンティティは必ず指定する必要があります。
  このアイデンティティは、明らかにHyperledger Fabricが許可型のネットワークであるということの基礎となるものです。
  アプリケーション、ピア、Orderer含め、全てのアクターはアイデンティティを持ち、それによってリソースに対する制御を決定されます。
  この考え方について詳しくは、メンバーシップサービスの[トピック](../membership/membership.html)を参照してください。


* `clientTlsIdentity`は、ウォレットから取得されるアイデンティ(**3a**)で、ゲートウェイと、ピアとordererといった様々なチャネルコンポーネントとのセキュア通信で使われます(**3b**)。

  このアイデンティティは、ユーザーのアイデンティティとは異なることに注意してください。
  `clientTlsIdentity`はセキュア通信では重要ですが、ユーザーのアイデンティティのように基礎をなすものではありません。それは、そのスコープがセキュアなネットワーク通信に限られているからです。

  `clientTlsIdentity`はオプショナルです。
  本番環境においてはセットすることをお勧めします。
  `clientTlsIdentity`と`identity`には常に異なるものを使用するべきです。これは、これらのアイデンティティが全く異なる意味とライフサイクルをもつからです。
  例えば、もし`clientTlsIdentity`が漏洩した場合には、`identity`もしてしまうことになるでしょう。
  二つを別々にしておくことによってよりセキュアになります。


* `eventHandlerOptions.commitTimeout`はオプショナルです。
  これは、ゲートウェイがアプリケーションに処理を返す前に、トランザクションがいずれかのピア(**4a**)でコミットされるまで待つ最大時間を秒単位で指定します。
  通知のために使うピアのセットは、`eventHandlerOptions.strategy`オプションで決定されます。
  もし、commitTimeoutが指定されない場合は、ゲートウェイは300秒のタイムアウトを使用します。


* `eventHandlerOptions.strategy`はオプショナルです。
  これは、ゲートウェイが、トランザクションがコミットされた通知を受け取るのに使うピアのセットを識別します。
  たとえば、一つのピアから受け取るのか、その組織のすべてのピアから受け取るのか、などです。
  以下のいずれかの値をとることができます。

  * `EventStrategies.MSPID_SCOPE_ANYFORTX`: その組織の**いずれかの**ピアから受け取ります。
    この例では、**4b**を参照してください。MagnetoCorpのPeer 1、Peer 2、Peer 3のどれでもゲートウェイに通知することができます。

  * `EventStrategies.MSPID_SCOPE_ALLFORTX`: **これがデフォルトの値です**。ユーザーの組織の**全ての**ピアから受け取ります。
    この例では、**4b**を参照してください。
    MagnetoCorpの全てのピア、すなわちPeer 1、Peer 2、Peer 3が全てゲートウェイに通知を行っていなければなりません。
    ピアは、知られている・ディスカバリで発見されている、かつ、利用可能なものだけが対象となります。
    停止していたり、エラーとなったピアは含まれません。

  * `EventStrategies.NETWORK_SCOPE_ANYFORTX`: ネットワークチャネル全体の**いずれかの**ピアから受け取ります。
    この例では、**4b**と**4c**を参照してください。
    MagnetoCorpからのPeer 1-3または、DigiBankのPeer 7-9のどれでもゲートウェイに通知することができます。

  * `EventStrategies.NETWORK_SCOPE_ALLFORTX`: ネットワークチャネル全体の**全ての**ピアから受け取ります。
    この例では、**4b**と**4c**を参照してください。
    MagnetoCorpとDigiBakの全てのピア、すなわちPeer1-3とPeer7-9がゲートウェイに通知しなければなりません。
    ピアは、知られている・ディスカバリで発見されている、かつ、利用可能なものだけが対象となります。
    停止していたり、エラーとなったピアは含まれません。

  * <`PluginEventHandlerFunction`>: ユーザー定義のイベントハンドラの名前です。
    これによって、ユーザーがイベント処理に自前のロジックを定義することができます。
    詳しくは、プラグインイベントハンドラの[定義](https://hyperledger.github.io/fabric-sdk-node/{BRANCH}/tutorial-transaction-commit-events.html)の方法や[サンプルハンドラ](https://github.com/hyperledger/fabric-sdk-node/blob/{BRANCH}/test/integration/network-e2e/sample-transaction-event-handler.js)を参照してください。

    ユーザー定義のイベントハンドラは、非常に独特なイベントハンドラの要件がある場合にのみ必要となります。
    一般的には、組み込みのイベントストラテジのいずれかで十分でしょう。
    ユーザー定義のイベントハンドラの例としては、トランザクションがコミットされたことの確認のために、ある組織の過半数のピアを待つといったものがあるでしょう。

    もし、ユーザー定義のイベントハンドラを指定しても、アプリケーションロジックには影響しません。
    この二つは切り離されたものです。
    ハンドラは処理の間にSDKによって呼ばれ、SDKはハンドラの結果によってどのピアをイベント通知に使うかを選びます。
    アプリケーションは、SDKがその処理を終えてから処理を受け取ります。

    もし、ユーザー定義のイベントハンドラが指定されなかった場合には、`EventStrategies`のデフォルト値が使われます。


* `discovery.enabled`はオプショナルで、`true`か`false`のいずれかの値をとります。
  デフォルトは、`true`です。
  これによって、ゲートウェイが[サービスディスカバリ](../discovery-overview.html)を使用して、コネクションプロファイルで指定されたネットワークトポロジを補うかどうかを決めます。
  図中の**6**を参照してください。ピアからのゴシップ情報をゲートウェイが使っています。

  この値は、`INITIALIZE-WITH-DISCOVERY`環境変数によって上書きされ、`true`か`false`を設定することができます。


* `discovery.asLocalhost`はオプショナルで、`true`か`false`のいずれかの値をとります。
  デフォルトは、`true`です。
  これによって、サービスディスカバリで得られたIPアドレスを、Dockerのネットワークからlocalhostに変換するかどうかを決定します。

  開発者は、ピア・orderer・CAといったネットワークコンポーネントとしてDockerコンテナを使うアプリケーションを書き、ただし、アプリケーション自身はコンテナで動かさないことが多いでしょう。
  これが、`true`がデフォルトである理由です。
  本番環境においては、アプリケーションはネットワークコンポーネントと同様にDockerコンテナで動作することが多いため、アドレス変換は必要ありません。
  この場合、アプリケーションは、明示的に`false`を指定するか、環境変数による上書きを使用しなければなりません。

  この値は、`DISCOVERY-AS-LOCALHOST`環境変数によって上書きされ、`true`か`false`を設定することができます。


## Considerations

コネクションオプションをどのように選ぶかを決める際には、以下の考慮すべき事項のリストが役に立つでしょう。

* `eventHandlerOptions.commitTimeout`と`eventHandlerOptions.strategy`は、組み合わせで意味を持ちます。
  たとえば、`commitTimeout: 100`と`strategy: EventStrategies.MSPID_SCOPE_ANYFORTX`は、ゲートウェイが*いずれかの*ピアがトランザクションがコミットされたことを最大で100秒まで待つ、ということを意味します。
  対照的に、`strategy:  EventStrategies.NETWORK_SCOPE_ALLFORTX`は、ゲートウェイは*全ての*組織の*全ての*ピアを100秒間まで待つということを意味します。


* デフォルトである`eventHandlerOptions.strategy: EventStrategies.MSPID_SCOPE_ALLFORTX`は、アプリケーションの組織の全てのピアがトランザクションをコミットするまで待ちます。
  これは、アプリケーションが、自分の組織のピアがすべて最新の台帳のコピーを持つことを確認でき、同時実行による問題を最小化することができるので、良いデフォルト値です。

  しかし、組織内のピア数が増えていくと、全てのピアを待つことは多少不要になってきます。
  この場合、プラグ可能なイベントハンドラによって、より効率的なストラテジを提供することができます。
  たとえば、合意形成によって全ての台帳が同期されるという想定に基づいて、トランザクションを送信するのと通知を受け取るのに、同じピアのセットを使うということができるでしょう。


* サービスディスカバリには、`clientTlsIdentity`が設定されている必要があります。
  これは、アプリケーションと情報を交換するピアが、情報を交換する相手が信頼している主体であると確信する必要があるためです。
  もし、`clientTlsIdentity`が設定されていなければ、`discovery`は、セットされているかに関わらず無視されるでしょう。


* アプリケーションは、コネクションオプションをゲートウェイに接続する際に指定することができますが、管理者によってこれらのオプションを上書きすることが必要になることがあります。
  これは、オプションが時間経過によって変化しうるネットワーク上でのやりとりにかかわるからです。
  例えば、管理者がサービスディスカバリによるネットワーク性能に対する影響を調べようとする場合です。

  よいアプローチとしては、アプリケーションがゲートウェイへの接続を設定するときに読む設定ファイルにアプリケーションごとの上書きを定義することです。

  ディスカバリに関するオプションである`enabled`と`asLocalhost`は最も頻繁に管理者によって上書きすることが必要になるため、`INITIALIZE-WITH-DISCOVERY`と`DISCOVERY-AS-LOCALHOST`環境変数が簡単のために提供されています。
  管理者は、アプリケーションの本番ランタイム環境ではこれらを設定すべきです。
  これは、Dockerコンテナであることが最も多いでしょう。

<!--- Licensed under Creative Commons Attribution 4.0 International License
https://creativecommons.org/licenses/by/4.0/ -->
