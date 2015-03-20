_Tutorial of bayon in Japanese_



# はじめに #

bayonは汎用的に利用できる[データクラスタリング](http://ja.wikipedia.org/wiki/データ・クラスタリング)ツールで、現在は Repeated Bisection法 と [K-means法](http://ja.wikipedia.org/wiki/K-means%E6%B3%95) に対応しています。

シンプルな構成で、かつ大規模なデータに対しても高速に実行できるところが特徴です。大量のデータを俯瞰したいときに、似た者同士のグループにサクッと分けて確認する、といったユースケースに利用できます。

またbayonが採用しているクラスタリング手法は、所属するクラスタが1つのみになるハードクラスタリングの手法ですが、別途各クラスタとドキュメントとの類似度を測定することで、所属度を持ちつつ複数のクラスタに所属するソフトクラスタリングと同等の結果を得ることができます。

# 更新情報 #
  * 2012/06/18: 0.1.1リリース
    * ビルドエラーが出ていたのを修正

  * 2010/09/15: 0.1.0リリース
    * リファクタリング

  * 2010/02/16: 0.0.10リリース
    * 一部の環境でmakeできない問題を修正しました

  * 2010/01/04: 0.0.9リリース
    * 前のバージョンがインストールされているときに、make checkが通らない問題を修正しました

  * 2009/12/24: 0.0.8リリース
    * **-C, --classify** オプション指定時のメモリ使用量を減少させました

  * 2009/08/12: 0.0.7リリース
    * 以前までのバージョンでは、入力された各ドキュメントのベクトルの要素を最大50個で制限していたのですが、デフォルトでは制限をなくし、全ての要素を使用するようにしました
    * **--vector-size** オプションで入力ドキュメントベクトルの最大要素数を指定できるようにしました
    * bug fix

  * 2009/07/17: 0.0.6リリース
    * google::dense\_hash\_mapがインストールされていないときにコンパイルエラーが発生してしまうのを解消

  * 2009/07/13: 0.0.5リリース
    * クラスタの中心ベクトルを出力するオプションを追加
    * idfオプションの追加
    * 乱数のseedをデフォルトでは固定にし、オプションで指定できるように変更
    * ドキュメントと各クラスタとの類似度を求めて、ソフトクラスタリングと同等の結果を出力できるオプションを追加

  * 2009/06/18: 0.0.4リリース
    * クラスタ中の各要素に対し、クラスタの中心との類似度を表示するオプションを追加
    * クラスタ中の各要素を出力する際に、クラスタの中心との類似度が高い順に出力するように変更
    * bug fix

  * 2009/06/10: 0.0.3リリース
    * Mac(OS X 10.5.7で検証)でコンパイルできるように変更

  * 2009/06/09: 0.0.2リリース

  * 2009/06/09: 0.0.1リリース

# ダウンロード #

[最新版のソースコード](http://code.google.com/p/bayon/downloads/list)をダウンロードしてください。

# インストール #

最新のソースコードをダウンロードして、以下のようにしてインストールしてください。make checkを実行する場合は [googletest](http://code.google.com/p/googletest/) が必要となるため、事前にgoogletestをインストールしておいてください。

また[google-sparsehash](http://code.google.com/p/google-sparsehash/)がインストールされていると、さらに高速に実行できるようになるため、事前にインストールしておくことをお勧めします。

```
% tar xvzf bayon-*.tar.gz
% cd bayon-*
% ./configure
% make
% make check  (googletest をインストールしておく必要があります)
% sudo make install
```

# ツールの使用方法 #

## 入力データのクラスタリング ##

クラスタリングを実行する場合は、以下のオプションを指定してコマンドラインツールを実行してください。クラスタリング結果は標準出力に出力されます。

各クラスタに含まれるドキュメントの並び順は、クラスタへの所属度(クラスタの中心ベクトルとのcosine類似度で-1〜1の値になります)が高い順にソートされています。デフォルトでは各ドキュメントのクラスタへの所属度は出力されませんが、 **-p, --point** オプションを指定すると所属度を含めてクラスタリング結果を出力します。

```
% bayon -n num [options] file
% bayon -l limit [options] file
   -n, --number=num      分割するクラスタの数
   -l, --limit=lim       分割ポイントの閾値
   -p, --point           各ドキュメントの所属クラスタへの所属度を出力
   -c, --clvector=file   クラスタの中心ベクトルの出力先ファイル
   --clvector-size=num   クラスタの中心ベクトルの最大要素数(デフォルトは50)
   --method=method       クラスタリング手法(rb, kmeans), デフォルトはrb
   --seed=seed           乱数のseed値を指定
```

### クラスタ数を指定して実行する場合 ###

出力するクラスタの数を指定する場合は、以下のように **-n, --number** オプションを指定して実行してください。

```
(100個のクラスタに分割)
% bayon -n 100 input.tsv > output.tsv

(各ドキュメントのクラスタへの所属度を加えて出力)
% bayon -n 100 -p input.tsv > output.tsv
```

### 分割ポイントを指定して実行する場合 ###

クラスタ数をどの程度にすれば適切に分割されるかを事前に予想することが難しい場合は、分割ポイントの閾値を設定して実行する方法を使用すると、ある程度適切な分割が完了するまで分割をしてくれます。入力データにもよりますが、ポイントは1.0〜2.0程度を指定するとよいです。

```
(分割ポイントの閾値が1.0を下回るまで分割)
% bayon -l 1.0 input.tsv > output.tsv
```

### クラスタの中心ベクトルを保存する場合 ###

各クラスタの中心ベクトルを保存する場合は、以下のように **-c, --clvector** オプションを指定して実行してください。オプションで指定されたファイルに、テキスト形式で中心ベクトルが出力されます。

このとき、 **--clvector-size** オプションを使用すると、各中心ベクトル中に含まれるキー・ポイントペアの最大数を指定することができます。デフォルトでは50個の要素が出力されます。

```
(centroid.tsvに中心ベクトルを出力)
% bayon -n 100 -c centroid.tsv input.tsv > output.tsv

(中心ベクトルの要素数を最大20個に指定)
% bayon -n 100 -c centroid.tsv --clvector-size 20 input.tsv > output.tsv
```

**-C, --classify** オプションを使用してドキュメントと各クラスタの類似度を求める場合は、この **-c, --clvector** オプションでクラスタの中心ベクトルを保存しておく必要があります。

### クラスタリング手法を変更する場合 ###

クラスタリング手法はデフォルトの設定では **rb** (Repeated Bisection法)を使用します。K-meansを使用してクラスタリングを行いたい場合は、以下のように **--method** オプションでクラスタリング手法を指定して実行してください。

```
(クラスタリング手法にK-meansを使用して、100個のクラスタに分割)
% bayon -n 100 --method kmeans input.tsv > output.tsv
```

通常使用する場合は、kmeansと比べてrbの方が高速でかつ精度もよいため、rbのまま実行することをおすすめします。

### 乱数のseed値を変更する場合 ###

bayonでは、クラスタリングを行う際に要素の選択をランダムに行っている箇所があり、そのため乱数のseed値を変更することで異なるクラスタリング結果を得ることができます。デフォルトではseed値は固定になっているため、毎回同じクラスタリング結果が得られます。

```
(乱数のseed値に123456を指定してクラスタリングを実行)
% bayon -n 100 --seed 123456 input.tsv > output.tsv
```

ただし、乱数を変更してもクラスタリング結果の精度にはそれほどの影響はないと思いますので、デフォルト設定での出力結果の精度に満足がいかなかった場合は、seedを変更するのではなく、入力データの精査を行った方がよいかもしれません。

## ドキュメントに類似するクラスタの特定 ##

上記のオプションで取得したクラスタリング結果は、各ドキュメントは1つのクラスタのみに所属します。ただ実際のデータでは、ただ1つのグループに属するのではなく、複数のグループに関連することがよくあります。そこでbayonでは、**-C, --classify** オプションを使用することで、各クラスタとドキュメントとの類似度を比較し、ドキュメントに関連するクラスタのリストを取得することができます。結果は標準出力に表示されます。

```
% bayon -C file [options] file
   -C, --classify=file   分類先のベクトル(クラスタの中心ベクトルなど)
   --inv-keys=num        入力ドキュメントのベクトルに含まれるキーの内、
                         転置インデックスを引くキーの最大数(デフォルトは20)
   --inv-size=num        転置インデックス中で各キーに対して保存しておく
                         要素の最大数(デフォルトは100)
   --classify-size=num   各ドキュメントに対する分類結果(類似しているグループ)
                         の最大数(デフォルトは20)
```

ただし、各クラスタとドキュメントとの類似度を求める場合は、事前にクラスタリングをする際に **-c, --clvector** オプションでクラスタの中心ベクトルを保存しておく必要がありますので、ご注意下さい。

```
(まずクラスタリングを行う。
 その際centroid.tsvにクラスタの中心ベクトルを保存しておく)
% bayon -c centroid.tsv -n 100 input.tsv > cluster.tsv

(クラスタの中心ベクトルとドキュメントのベクトルを比較)
% bayon -C centroid.tsv input.tsv > classify.tsv
```

また実際には、 **-C, --classify** オプションで指定するベクトルは、クラスタの中心ベクトル以外の任意のベクトルが使用できます。そのため、単にドキュメント間の類似度を調べたい、といった用途にも利用できます。

### 転置インデックス用オプション ###

**-C, --classify** オプションで類似するベクトルを調べるとき、システムの内部では **-C, --classify** オプションで指定されたベクトル集合の[転置インデックス](http://ja.wikipedia.org/wiki/%E8%BB%A2%E7%BD%AE%E3%82%A4%E3%83%B3%E3%83%87%E3%83%83%E3%82%AF%E3%82%B9)を構築し、計算の高速化を行っています。この転置インデックス用のオプションを変更することで、 **-C, --classfiy** オプション使用時の実行時間の短縮もしくは精度の向上を行える場合があります(入力データによっては変化がないかもしれません)。

  * **--inv-keys=num** : ドキュメントとの類似度を計算する時に、ドキュメントベクトル中のキーの内、転置インデックスを引くキーの数を指定する
  * **--inv-size=num** : 転置インデックス構築時に、各キーが保持する関連クラスタの個数

ただし通常は上記オプションは指定せず、デフォルト設定のままで問題ないと思います。

## 共通オプション ##

以下のオプションは、クラスタリング、類似クラスタ特定の両方で使用できます。

```
   --vector-size         入力されたドキュメントベクトルの最大要素数を指定
   --idf                 入力されたドキュメントベクトルに対しIDFを適用
   -h, --help            ヘルプメッセージを表示
   -v, --version         バージョン情報を表示
```

**--vector-size** オプションを指定すると、入力された各ドキュメントベクトル中の要素のうち、ベクトル比較に使用する要素の最大数を指定できます。デフォルトではすべての要素を使用します。このオプションを指定することで、システムの実行時間の短縮や、精度の向上を行える場合があります。

**--idf** オプションについて詳しくは、下記の入力データフォーマットの節をご参照下さい。

# 入力データのフォーマット #

## ドキュメントのリスト ##

クラスタリング・類似クラスタ特定の対象となるドキュメントリストのデータは、以下のようにタブ区切りのフォーマットのテキストファイルにしてください。1行に1つのドキュメントの情報を記入し、空行は含めないようにしてください。各行は先頭にそのドキュメントのIDを記入し、IDの後はそのドキュメントの特徴を表すキーとそのポイントを記入してください。キーとポイントのペアは複数記入できますが、必ずペアで入力する必要があります。

```
document_id1 \t key1-1 \t value1-1 \t key1-2 \t value1-2 \t ...\n
document_id2 \t key2-1 \t value2-1 \t key2-2 \t value2-2 \t ...\n
...
```

各フィールドは以下を表しています。

  * _document\_id_: 各ドキュメントのIDで、ドキュメント毎にユニークである必要があります。タブを含まない任意の文字列を使用できます。
  * _key_: ドキュメントの特徴を表す文字列です。タブを含まない任意の文字列が使用できます。
  * _value_: keyの特徴の度合いを表し、この値が高いほどより特徴的なフレーズになります。実数が使用できます。

valueは特徴の度合いを正しく反映している必要があります。そのため、例えば文書集合のクラスタリングを行う場合等では、単に単語の頻度を使うのではなく、[tf-idf](http://ja.wikipedia.org/wiki/Tf-idf)などを使用して、より適切にポイントづけを行うことで、クラスタリングの精度を上げることができます。以下で述べるように、bayonでは **--idf** オプションを使用することで特徴的なキーを自動判定することができますので、ツールユーザ側で特徴的なキーを求めることが難しい場合は下記オプションをご利用下さい。

### idfオプションで特徴的なキーを自動判定する ###

クラスタリング・分類を実行する際に **--idf** オプションを使用すると、入力データの各キーに対し、特徴的なキーのポイントを上げ、逆にあまり特徴的ではないキーのポイントを下げてから、クラスタリング・分類を実行するようになります。

これによりクラスタリング・分類の精度向上が見込めます(データにはよっては精度が下がる場合もありえます)。またツールユーザが入力データを用意する際に、ユーザ側ではキーの特徴度合いを調べる必要がなくなり、単にキーの頻度を数えただけのデータでも **--idf** オプションを使用することで、高い精度の結果を得ることが可能になります。

idfの計算方法について詳しく知りたい方は、wikipediaの[tf-idf](http://ja.wikipedia.org/wiki/Tf-idf)のページをご参照下さい。

## クラスタの中心ベクトルのリスト(入力) ##

**-C, --classify** オプションを使用して各ドキュメントに類似するクラスタを特定するとき、 **-C, --classify** オプションに与えるクラスタの中心ベクトルのリストは、以下のようにタブ区切りのテキストフォーマットにしてください。各フィールドはドキュメントのリストデータと同じです。

```
cluster_id1 \t key1-1 \t value1-1 \t key1-2 \t value1-2 \t ...\n
cluster_id2 \t key2-1 \t value2-1 \t key2-2 \t value2-2 \t ...\n
...
```

通常は **-c, --clvector** オプションによりシステムが自動的にデータを生成しますので、そちらを使用してください。

## 入力データの例 ##

特徴として、よく聞く音楽のジャンルを使った入力例を示します。

```
阿佐田   J-POP       10   J-R&B       6   ロック  4
小島     ジャズ       8   レゲエ      9
古川     クラシック   4   ワールド    4
田村     ジャズ       9   メタル      2   レゲエ  6
青柳     J-POP        4   ロック      3   HIPHOP  3
三輪     クラシック   8   ロック      1
```

上記の例では「阿佐田」「小島」「古川」が各ドキュメントのID(人名)を表し、「阿佐田」の特徴を表すフレーズ(音楽のジャンル)として「J-POP」「J-R&B」「ロック」が挙げられています。

クラスタの中心ベクトルデータの例は、出力データ例の節で説明します。

# 出力データのフォーマット #

## クラスタのリスト ##

クラスタリング結果は、以下のようなタブ区切りのフォーマットのテキストになります。1行が1つのクラスタを表し、先頭がクラスタのID、それ以降はそのクラスタに所属するドキュメントのIDをタブ区切りでつなげたリストになります。

```
cluster_id1 \t document_id1 \t document_id2 \t document_id3 \t ...\n
cluster_id2 \t document_id4 \t document_id5 \t document_id6 \t ...\n
...
```

## クラスタのリスト(各ドキュメントのクラスタへの所属度も表示) ##

クラスタリングを実行する際に **-p, --point** オプションを使用すると、上記のクラスタリストに加えて、各ドキュメントの所属クラスタへの所属度を表示します。この所属度はドキュメントのベクトルと、クラスタの中心ベクトル間のcosine類似度で計算しており、-1.0〜1.0の値を取ります。

```
cluster_id1 \t document_id1 \t point1 \t document_id2 \t point2 \t ...\n
cluster_id2 \t document_id3 \t point3 \t document_id4 \t point4 \t ...\n
...
```

## クラスタの中心ベクトルのリスト(出力) ##

クラスタリングを実行する際に **-c, --clvector=file** オプションを使用すると、指定した出力ファイルにクラスタの中心ベクトルのリストが出力されます。出力データのフォーマットは以下のようなタブ区切りのテキストになり、1行が1つのクラスタを表し、先頭がクラスタのID、それ以降は中心ベクトルになります。

```
cluster_id1 \t key1-1 \t value1-1 \t key1-2 \t value1-2 \t ...\n
cluster_id2 \t key2-1 \t value2-1 \t key2-2 \t value2-2 \t ...\n
...
```

## 各ドキュメントに類似するクラスタのリスト ##

**-C, --classify** オプションを使用して、各ドキュメントに対し類似しているクラスタを求めたときの結果は、以下のタブ区切りのテキストフォーマットになります。1行が1ドキュメントの情報を表し、先頭がドキュメントのID、それ以降は関連するクラスタのIDとその類似度になります。:

```
document_id1 \t cluster_id1 \t point1 \t cluster_id2 \t point2 \t ...\n
document_id2 \t cluster_id3 \t point3 \t cluster_id4 \t point4 \t ...\n
...
```

## 出力データの例 ##

上記の音楽のジャンルを使用した例を3つのクラスタに分割したときの出力は、以下になります。

```
% bayon -n 3 data/test2.tsv
1       田村    小島
2       阿佐田  青柳
3       三輪    古川
```

**-p, --point** オプションでクラスタへの所属度も表示した場合は、以下のような出力になります。

```
% bayon -n 3 -p data/test2.tsv
1       田村    0.987737        小島    0.987737
2       阿佐田  0.928262        青柳    0.928262
3       三輪    0.922401        古川    0.922401
```

このクラスタの中心ベクトルは以下になります。

```
% bayon -n 3 -c centroid.tsv data/test2.tsv > cluster.tsv
% cat centroid.tsv
1   ジャズ  0.750476    レゲエ  0.654458    メタル  0.0920378
2   J-POP   0.806401    ロック  0.451887    HIPHOP  0.277129    J-R&B   0.262137
3   クラシック  0.921175    ワールド    0.383297    ロック  0.0672347
```

ドキュメントと各クラスタの類似度を求めた結果は以下になります。

```
% bayon -C centroid.tsv data/test2.tsv
阿佐田  2       0.928261        3       0.0218138
小島    1       0.987737
古川    3       0.922401
田村    1       0.987737
青柳    2       0.928262        3       0.034592
三輪    3       0.922401        2       0.0560497
```

# 実行時間 #

いくつかデータセットを用意し、それらに対しbayonでクラスタリングと類似クラスタ特定を実行したときの実行時間を以下に示します。クラスタリング・類似クラスタ特定ともに高速に実行できています。

| **データ数** | **クラスタ数** | **クラスタリング(秒)** | **類似クラスタ特定(秒)** |
|:-----------------|:--------------------|:-------------------------------|:----------------------------------|
| 1000 | 10 | 0.17 | 0.088 |
| 50000 | 1000 | 36 | 13.4 |
| 500000 | 10000 | 720 | 1385 |

# クラスタリング手法の説明 #

## Repeated Bisection ##

bayonでは、Repeated Bisectionと呼ばれるクラスタリング手法を採用しています。Repeated Bisectionはクラスタリングツール[CLUTO](http://glaros.dtc.umn.edu/gkhome/views/cluto)で使用されているクラスタリング手法で、データ集合を繰り返し2分割することでクラスタリングを実行します。K-means法などと比較して、高速に実行でき、また精度も良好なようです。

より詳しく説明しますと、 Repeated Bisectionは以下の1-4の処理を実行し、繰り返しクラスタを2分割していくことでクラスタリングを行います。


  1. 分割するクラスタを1つ選択する(一番クラスタ内のまとまりが悪いものを選択)
  1. クラスタ中からランダムに2つ要素を選択し、それぞれが格納したクラスタを2つ作成する
  1. 元のクラスタ中の全ての要素に対し、2で選んだ要素との類似度を求め、類似度が高い方のクラスタに要素を追加する
  1. 2クラスタ間で要素の移動を行い、分割結果の洗練を行う(移動できる要素がなくなるまで続ける)

![http://mahjong-mania.net/bayon/img/rb.png](http://mahjong-mania.net/bayon/img/rb.png)

ここで、2-4のプロセスだけに注目してみると、これはクラスタの中心を2つとしてK-means法を実行しているのと同じことをしています。つまりRepeated BisectionはK-meansを繰り返し実行しているだけなのです。

プロセス1でまとまりの悪いクラスタを選択するとき、プロセス4で要素を移動してクラスタの洗練を行うときは、クラスタのまとまり具合を評価する必要があります。このクラスタの評価は、クラスタの各要素とクラスタの中心とのcosine類似度の和としています。この和が大きいほどクラスタの中心に凝集しているようになり、クラスタ中のデータが似た者同士の集まりである、と考えるわけです。ただし、クラスタ中の各要素と中心との類似度をまじめに全て比較するのは計算量も高くなってしまいます。実際には、この値はクラスタ中のベクトルをすべて足した複合ベクトルの長さと同値になり、そのため計算量も少なく評価値を求めることができます。

詳しくは[CLUTO](http://glaros.dtc.umn.edu/gkhome/views/cluto)のサイトから論文を参照してください。

## K-means ##

K-meansについては、Wikipediaの[K-means法のページ](http://ja.wikipedia.org/wiki/K-means%E6%B3%95)など、説明をしているwebページがたくさんあるのでそちらをご参照下さい。

# 関連ページ #

  * [mixi Engineer's Blog - 軽量データクラスタリングツールbayon](http://alpha.mixi.co.jp/blog/?p=1049)
  * [mixi Engineer's Blog - bayonでソフトクラスタリング](http://alpha.mixi.co.jp/blog/?p=1245)