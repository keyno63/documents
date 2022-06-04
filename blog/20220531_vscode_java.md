# はじめに
## 動機
私は普段Javaなどの開発をするとき、[IntelliJ IDEA Ultimate]を使って開発しています[^1]。

しかし、私の周りの環境では[Visual Studio Code] (VSCode)を使われるケースが多く、

Javaの開発もVisual Studio Codeで整えたいという要望があるのですが、

私に知見がなく、説明するのが困難で合ったため、

一度真剣に開発環境を構築し、その時の情報を保存しておこうとしました。

## Visual Studio Code が望まれる理由

Visual Studio Code を使う場合のメリットについても少し考えてみました。

これは私の観測範囲ベースも含まれていますが、

Visual Studio Codeの長所であったり、

こちらの方が（少なくとも IntelliJ IDEA よりも）望まれている理由としては以下があります。

- 豊富や拡張パックで柔軟に機能をカスタマイズできる
- 他言語の開発を扱う現場で、ワンツールで完結する
- 無料で使うことができる

もしこの中で魅力に感じる部分があれば

Visual Studio CodeでのJava開発環境に挑戦して見ても良いかもしれません。

# Visual Studio Code 設定
## Visual Studio Code のインストール
以下のダウンロードサイトから環境に合った圧縮ファイル、またはインストーラーをダウンロードし、  
開発環境にインストールします。

[Visual Studio Code ダウンロードサイト]

- Windows の場合、インストーラー、またはzipファイルをダウンロードします。  
  インストーラーからインストールする場合は、手順に従ってインストールしてください。
- zipファイルをダウンロードし、解凍すると実行ファイルが含まれています。  
  実行ファイルを適切な場所に移動すると良いと思います。  
  Mac の場合、必要に応じてAppにコピーします。

## JDK のインストール

JDKをダウンロードし、開発環境から使用できるように準備が必要です。

Java は複数の組織がバイナリを提供しているため、提供元が複数存在します。

Java は互換性があるため、多くの場合では違いを意識する必要はありません、

環境毎で指定のJDKがあれば、指定のものをダウンロードして利用してください。

特に指定やこだわりがなければ Oracle、もしくは Adoptium あたりを使われる場合が多い印象があります。

### SDKMAN を用いたインストール

[SDKMAN] を用いてJDKのインストールすることが可能です。

Windows 以外であればこの方法が一番おすすめです。

ディストリビューションの一覧やダウンロードまで実施できますし、

利用に必要な設定や管理も行えます。

SDKMANのインストール方法・使用方法は以下を参考にしてください。

[SDKMAN インストール方法]

SDKMAN を用いたJDKのインストールの例を以下に示します（java.net OpenJDK 18 の場合）。

```shell
sdk install java 18.0.1.1-open
```

実行すると、以下のような出力がされ、インストールが実行されていきます。
```
Downloading: java 18.0.1.1-open

In progress...

################################################################################################################### 100.0%

Repackaging Java 18.0.1.1-open...

Done repackaging...
Cleaning up residual files...

Installing: java 18.0.1.1-open
Done installing!

Do you want java 18.0.1.1-open to be set as default? (Y/n): Y

Setting java 18.0.1.1-open as default.
```

### Java Extension Pack の Getting Start の機能からダウンロードする

後述する「Java Extension Pack」インストール後であれば、Visual Studio Code 上の操作から JDK をダウンロードすることが可能です。

Java Extension Pack の「Get Start with Java Development」の機能からもJDKをインストールすることができます。

Visuial Code Studio を開き、「Help」->「Get Start」を選択します。

「Walkthroughs」の「More...」をクリックし、「Open Walkthroughs...」という検索ボックスが出るので「Get STarted with Java Development」を入力し実行する。

[f:id:keyno63:20220604122659p:plain]

「Get STarted with Java Development」の画面に遷移するので、「Get your runtime ready」という項目から「Install JDK」をクリックします。

「Install New JDK」という画面が表示されるので、必要なJDKをインストールすることができます。

### 各JDKディストリビューションのダウンロードサイトから取得する

各JDKのディストリビューションのダウンロードサイトから取得する方法があります。

何箇所かのディストリビューションとダウンロード先は以下になります。

- [Oracle OpenJDK](https://jdk.java.net/)
- [RedHat OpenJDK](https://developers.redhat.com/products/openjdk/download)
- [Azul Zulu](https://www.azul.com/downloads/)
- [Amazon Corretto](https://aws.amazon.com/jp/corretto/)
- [Adoptium](https://adoptium.net/temurin/releases/)(旧Adopt)

## Visual Studio Code の設定
### 拡張パックの追加

Javaの開発で必要となる拡張パックをインストールします。

必要となる主な拡張パックは以下になります。

| 拡張パック名                                 | パッケージURL                                                                         | 概要                                                                                                                |
|----------------------------------------|----------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| Extension Pack for Java                | https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-pack     | Java の開発用拡張パックです。<br/>Javaのコード補完、ビルド、GUIからのJUnit（ユニットテスト）実行などが可能になります。                                            |
| Spring Boot Extension Pack             | https://marketplace.visualstudio.com/items?itemName=Pivotal.vscode-boot-dev-pack | SpringBoot 開発用拡張パックです。<br/>SpringBoot由来のアノテーションの解決や機能と参照できるようになります。<br/>SpringBootを使用しない開発では不要です。                 |
| Lombok Annotations Support for VS Code | https://marketplace.visualstudio.com/items?itemName=GabrielBB.vscode-lombok      | Lombok 用の拡張パックです。<br/>`@Getter`, `@Setter`, `@Slf4j`などのLombok由来のアノテーションの解決ができるようになります。<br/>Lombokを使用しない開発では不要です。  |

## Java用の設定追加

設定ファイルにJavaを実行するための設定を追加します。

setting.json にJDKへのパスを指定する `java.jdt.ls.home` を設定します[^2]。

```json
{
  "java.jdt.ls": "/Users/<user_name>/.sdkman/candidates/java/current/"
}
```
<user_name> は環境に応じて読み替えてください。

`java.home` のパス自体も環境に応じて変更してください。

`settings.json` を表示させる方法は設定画面から、右上の方にある「Open Settings」の画像をクリックして表示することができます
[f:id:keyno63:20220531012008p:plain]

設定画面の出し方は以下のいずれか

| OS      | 画面での操作                             | ショートカットキー     |
|---------|------------------------------------|---------------|
| Mac     | 「Code」-> 「Preferences」->「Settings」 | 「Command」+「,」 |
| Windows | 「File」-> 「Preferences」->「Settings」 | 「Ctrl」+「,」    |

## 設定の完了

以上で設定は完了です。

開発が必要なレポジトリーを使って確認してみてください。

開発対象のレポジトリーがない、Javaプロジェクトの作成方法がピンとこない。という方がいましたら、

動作確認用に以下のレポジトリーを用意しましたので、お試しください。

[Java-Gradle の動作練習用レポジトリ]

ご参考までに。

# 最後に
## 結び
Visual Studio Code でJavaの開発ができるようになりました。

開発ツール自体の初期設定やJDKのダウンロードなどが必要でしたが、

手順を進めれば開発環境が作れることが実証できました。

## 参考

- [Visual Studio java-tutorial]

[^1]: IntelliJ IDEA のインストール、設定方法。および使い方が気になる方は [プロになるJava](https://www.amazon.co.jp/dp/B09VK3FTDM) を参考にしてみてください
[^2]: 古いバージョンだと "java.home" になりますが、現行（2022年6月時点）では非推奨になっています。

[IntelliJ IDEA Ultimate]: https://www.jetbrains.com/ja-jp/idea/
[Visual Studio Code]: https://code.visualstudio.com/
[Visual Studio Code ダウンロードサイト]: https://code.visualstudio.com/download
[SDKMAN]: https://sdkman.io/
[SDKMAN インストール方法]: https://sdkman.io/install
[Java-Gradle の動作練習用レポジトリ]: https://github.com/keyno63/java-gradle-sandbox-project
[Visual Studio java-tutorial]: https://code.visualstudio.com/docs/java/java-tutorial
