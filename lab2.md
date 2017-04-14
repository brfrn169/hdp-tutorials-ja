
[LAB 2 - HIVE AND DATA ETL](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/#section_4)の翻訳です。

# LAB 2 - HIVE AND DATA ETL

## INTRODUCTION

このチュートリアルでは、Apache（TM）Hiveについて紹介します。前のセクションでは、HDFSにデータをロードする方法について説明しました。これで、geolocationとtrucksがcsvファイルとしてHDFSに保存されました。このデータをHiveで使用するために、テーブルを作成する方法とHiveウェアハウスに移動する方法を説明します。Hive User ViewsでSQLを使用してこのデータを分析し、それをORCとして保存します。Apache TezとそのDAGの作成方法についても説明します。それでは始めましょう..！！

## PRE-REQUISITES

このチュートリアルは、Hortonworks Sandboxを使用してHDPを始めるためのシリーズの一部です。このチュートリアルを進める前に、前提条件を満たしていることを確認してください。

* [Learning the Ropes of the Hortonworks Sandbox](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/ "Learning the Ropes of the Hortonworks Sandbox")
* Hortonworks Sandbox
* Lab 1: Load sensor data into HDFS

このチュートリアルを完了するのに1時間ほどを要します。

## OUTLINE

* Apache Hive basics
* Step 2.1: Become Familiar with Ambari Hive View
* Step 2.2: Define a Hive Table
* Step 2.3: Explore Hive Settings on Ambari Dashboard
* Step 2.4: Analyze the Trucks Data
* Step 2.5: Define Table Schema
* Summary
* Suggested readings

## APACHE HIVE

Apache Hiveは、Hadoopと統合されたさまざまなデータベースやファイルシステムに格納されたデータを照会するためのSQLインターフェースを提供します。Hiveは、SQLに慣れ親しんだアナリストが大量のデータに対してクエリを実行できるようにします。Hiveの主な3つの機能は、データの要約、クエリ、分析です。また、Hiveは、ETL(Extraction, Transformation, Loading)を可能にするツールを提供します。

## STEP 2.1: BECOME FAMILIAR WITH AMBARI HIVE VIEW

Apache Hiveは、HDFS内のデータのリレーショナル・ビューを提示します。Hiveは、データが格納されているファイル形式にかかわらず、Hiveで管理されている表形式またはHDFSに格納されている形式でデータを表現できます。また、Hiveは、RCFileやテキストファイル、ORC、JSON、parquet、sequence file、その他の多くの形式のデータを表形式で照会できます。SQLを使用することで、データを表として見ることができ、RDBMSのようにクエリを作成できます。

Hiveとのやり取りを容易にするため、Hortonworks SandboxではAmbari Hive Viewというツールを使用しています。Ambari Hive Viewは、Hiveへのインタラクティブなインターフェイスを提供します。クエリを作成、編集、保存、実行することができ、MapReduceジョブやTezジョブを用いてHiveのクエリが実行されます。

それではAmbari Hive Viewを開き、その環境を体験してみましょう。9つの四角のボタンをクリックし、Hive Viewを選択します。

![Hive  View](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/hive_view_lab2.png)

Ambari Hive Viewは以下のようになります。

![Hive  View](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/ambari_hive_user_view_concepts.png)

それでは、Hive ViewのSQL編集機能を詳しく見ていきましょう。

1. SQLインタラクションのための5つのタブ
  * Query：これは上の図に示したメインとなるインターフェイスであり、新しいSQLの記述、編集、実行のためのインタフェースです。
  * Saved Queries：Hive Viewではお気に入りのクエリを保存することができます。そのクエリをすぐにアクセスして再実行または編集することができます。
  * History：過去のクエリや現在実行中のクエリが表示され、それに対して編集や再実行することができます。また、参照権限を持つすべてのSQLを表示することもできます。たとえば、オペレータとアナリストがクエリのヘルプを必要とする場合、Hadoopオペレータはヒストリ機能を使用してレポートツールから送信されたクエリを表示できます。
  * UDFs：UDFインターフェースとその関連クラスを定義して、SQLエディターからアクセスできるようにします。
  * Upload Table：指定のデータベースにHiveテーブルをアップロードすることができます。アップロードされたテーブルは、クエリエディタ上ですぐに見ることができます。
2. Database Explorer：データベースオブジェクト(Database, Table, Column)をナビゲートできます。データベースオブジェクトを探すためには検索ボックスを使うか、ナビゲーション・ペインのDatabase -> Table -> Columnsを選択します。
3. Query Editor： SQLの作成および編集するためのペインです。このエディタには、クエリーの作成に役立つCTRL + Spaceによるコンテンツアシストが含まれています。
4. SQLを作成したら、次の4つのアクションを選択できます。
  * Execute：SQLを実行します。
  * Explain：SQLがどのように実行されるかを視覚的に確認することができます。
  * Save as...：クエリを保存することができます。
  * Kill Session：SQLの実行を終了します。
5. クエリが実行されると、ログまたは実際のクエリの結果が表示されます。
  * Logs Tab：クエリが実行されるとそのクエリに関連したログを見ることができます。もしクエリが失敗した時は、ここからトラブルシューティングのための情報を得ることができます。
  * Results Tab：デフォルトで50の結果セットが表示されます。
6. 右側に5つのスライディングビューがあり、以下の機能があります。
  * Query：これはデフォルトのビューで、SQLの作成や編集ができます。
  * Settings：グローバルまたは個々のクエリに対してプロパティ(設定)をセットすることができます。
  * Data Visualization：数値データをさまざまなグラフで視覚化することができます。
  * Visual Explain：クエリの実行計画が表示されます。また、クエリの進行状況も表示されます。
  * TEZ：TEZをクエリ実行エンジンとして使用すると、クエリに関連付けられているDAGを表示できます。これはTEZ User Viewが統合され、SQLクエリに関連付けられたTEZジョブを視覚化することによって、正確性をチェックし、パフォーマンスチューニングに役立ちます。
  * Notifications：クエリの実行に関するフィードバックを得る方法を指定できます。

実際に、この様々なHive Viewの機能を調べてみましょう。

### 2.1.1 Set hive.execution.engine as Tez

Hiveのクエリを実行する前に、実行エンジンをTezとして設定します。MapReduceを使用することもできますが、このチュートリアルではTezを使用します。

1. 上の画像の6のサイドバーのギアアイコンをクリックします。

2. ドロップダウンメニューをクリックし、hive.execution.engineを選択し、値をtezとして設定します。これで、このチュートリアルのクエリを実行する準備が整いました。

## STEP 2.2: DEFINE A HIVE TABLE

もう既にあなたはHive Viewに慣れているので、geolocationとtrucksのテーブルを作成し、それらのデータをロードしてみましょう。このセクションでは、Ambari Hive Viewを使用して2つのテーブルを作成する方法を学習します。Upload Tableタブには、入力ファイルの種類、ストレージオプション（ORCなど）、最初の行をヘッダーとして設定するなどの主要なオプションがあります。下記の図は、テーブルと次のステップから行うデータロードプロセスを視覚的に表しています。

![Load Creation Process ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/create_tables_architecture_lab2.png)

### 2.2.1 Create and load Trucks table For Staging Initial Load

Ambari Hive Viewの`Upload Table`タブを選択します。次に、`Upload from HDFS`のラジオボタンを選択し、HDFSパス`/user/maria_dev/data/trucks.csv`を入力し、`Preview`ボタンをクリックします。

![Upload Table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/upload_table_hdfs_path_lab2.png)

下記のようなダイアログが表示されます。最初の行には列の名前が含まれています。

![Upload Table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/click_gear_button_lab2.png)

幸いにも、`Upload Table`タブには、列名のヘッダーとして最初の行を指定する機能があります。上記の`File type`のプルダウンメニューの横にあるギアボタンを押して、ファイルタイプのカスタマイズウィンドウを開きます。 次に、`Is first row header?`チェックボックスをオンにし、closeボタンを押してください。

![Upload Table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/first_row_header_lab2.png)

そうすると下記のようなダイアログボックスが表示され、列の名前としてヘッダー列の名前が表示されます。

![Upload Table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/upload_table_lab2.png)

それらの設定が完了したら、`Upload Table`ボタンを選択して、表の作成およびロード処理を開始します。

![Upload Table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/upload_progress_lab2.png)

アップロードの進捗状況で何が起こっているのかを確認する前に、Hiveのファイル形式について学びましょう。

### 2.2.2: Define an ORC Table in Hive Create table using Apache ORC file format

[Apache ORC](https://orc.apache.org/)は、Hadoopのワークロードのための高速なカラムナファイルフォーマットです。

Optimized Row Columnar（[new Apache ORC project](http://hortonworks.com/blog/apache-orc-launches-as-a-top-level-project/)）ファイル形式は、Hiveデータを効率的に格納する方法を提供します。他のHiveファイル形式の限界を克服するように設計されています。ORCファイルを使用すると、Hiveがデータの読み取り、書き込み、および処理中にパフォーマンスが向上します。

ORCを使用するには、表を作成するときにファイル形式としてORCを指定します。以下、例になります。

```
CREATE TABLE … **STORED AS ORC**
CREATE TABLE trucks STORED AS ORC AS SELECT * FROM trucks_temp_table;
```

ORCを指定したcreate文は`Upload Table`タブの処理中でも、一時テーブルと共に使われています。

### 2.2.3: Review Upload Table Progress Steps

`trucks.csv`のテーブルは、最初に一時テーブルとして作成され、そこにデータがロードされます。 一時テーブルは、ORC形式でデータを作成およびロードするために使用されます。データが最終テーブルにロードされると、一時テーブルが削除されます。

![Upload Table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/create_tables_architecture_lab2.png)
注：一時テーブル名はランダムな文字セットであり、上の図の名前ではありません。

`History`タブを選択し、`Upload Table`タブを使用した結果として実行された4つの内部ジョブをクリックすると、発行されたSQL文を確認できます。

![Upload Table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/job_history_lab2.png)

### 2.2.4 Create and Load Geolocation Table

ORCファイル形式のgeolocationテーブルを作成して読み込むために、上記の手順を`geolocation.csv`で繰り返します。

### 2.2.5 Hive Create Table Statement

上記で生成され発行されたCREATE TABLEステートメントのいくつかの側面を見てみましょう。SQLのバックグラウンドを持っているなら、このステートメントはカラム定義の後ろの最後の3行を除いて非常によく似ていることがわかるはずです。

* ROW FORMAT句は、各行が改行文字で終了することを指定します。
* FIELDS TERMINATED BY句は、テーブルに関連付けられたフィールドはコンマで区切られていることを指定しています。
* STORED AS節は、表がTEXTFILE形式で保管されることを指定します。

注：これらの詳細については、[Apache Hive Language Manual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)を参照してください。

### 2.2.6 Verify New Tables Exist

テーブルが正常に定義されたことを確認するには、Database Explorerで`refresh`アイコンをクリックします。 Databaseでdefault databaseをクリックしてテーブルの一覧を展開し、新しいテーブルが表示されます。

![Verify Table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/select_data_trucks_lab2.png)

### 2.2.7 Sample Data from the trucks table

`Load sample data`アイコンをクリックして、100行を問い合わせるselect文を生成し実行します。

* 各エディタワークシート内に複数のSQL文を入れることができますが、各文をセミコロン";"で区切る必要があります。
* ワークシート内に複数のステートメントがあり、そのうちの1つだけを実行したい場合は、実行するステートメントを強調表示してから、Executeボタンをクリックします。

#### A few additional commands to explore tables

* `show tables;` – HCatalogに格納されているメタデータからテーブルのリストを参照して、データベースに作成されたテーブルを一覧表示します。
* `describe {table_name};` – 特定のテーブルのカラムのリストを表示します。 (例： `describe geolocation_stage;`)
* `show create table {table_name};` – テーブルを作成するためのDDLを表示します。 (例： `show create table geolocation_stage;`)
* `describe formatted {table_name};` – テーブルに関する追加のメタデータを表示します。たとえば、geolocationがORCテーブルであることを確認するには、次のクエリを実行します。
```
describe formatted geolocation;
```
Resultsタブの一番下までスクロールすると、Storage Informationというラベルのセクションが表示されます。出力は次のようになります。
![Output of formatted describe ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/storage_information_lab2.png)

デフォルトでは、Hiveでテーブルを作成すると、テーブルと同じ名前のディレクトリがHDFSの`/apps/hive/warehouse`フォルダに作成されます。Ambari Files Viewを使用して、`/apps/hive/warehouse`フォルダに移動します。 `geolocation`と`trucks`ディレクトリの両方が表示されます。

注：Hiveテーブルとそれに関連するメタデータの定義（つまり、データが格納されているディレクトリ、ファイル形式、Hiveプロパティが設定されているものなど）は、Hiveメタストアに格納されます。Hiveメタストアは、SandboxではMySQLデータベースを使っています。

### 2.2.8 Rename Query Editor Worksheet

新しいワークシートのタブに`trucks sample data`が表示されていることに注意してください。ワークシート・タブをダブルクリックして、ラベルの名前を"sample truck data"に変更します。このワークシートを保存するには、そのボタンをクリックします。

![Rename Worksheet ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/save_truck_sample_data_lab2.png)

### 2.2.9 Command Line Approach: Populate Hive Table with Data

次のHiveコマンドを使用して、コマンドラインから既存のテーブルにデータをロードすることができます。

```
LOAD DATA INPATH '/user/maria_dev/data/trucks.csv' OVERWRITE INTO TABLE trucks;
```

上記のコマンドを実行し、`/user/maria_dev/data`フォルダに移動したら、あなたは、フォルダが空であることに気付くでしょう！ `LOAD DATA INPATH`コマンドは、`trucks.csv`ファイルを`/user/maria_dev/data`フォルダから`/apps /hive/warehouse/trucks_stage`フォルダに移動させたのです。

### 2.2.10 Beeline – Command Shell

ここまで出てきたようなSQLをコマンドラインから実行したい場合は、Beeline Shellを使用できます。BeelineはJDBC接続を使用してHiveServer2に接続します。以下の手順をシェル（またはWindowsを使用している場合はputty）から実行します。

1. LocalのSandbox VMにsshで接続する
```
ssh maria_dev@127.0.0.1 -p 2222 maria_dev
```
2. `su hive`
3. `beeline`
  * Beeline Shellを開始し、コマンドとSQLを入力できるようになります
4. `quit;`
	* Beeline Shellから抜けます

ShellからHiveクエリを実行した場合のパフォーマンスについて

* Shellで実行する場合は、Ambari Hive Viewでの実行より高速になります。Shellで実行する場合はHiveクエリをHadoopをダイレクトに実行するのに対して、Ambari Hive ViewではクエリをHadoopに送信する前にRestサーバーを経由する必要があるためです。
* [Beeline from the Hive Wiki](https://cwiki.apache.org/confluence/display/Hive/HiveServer2+Clients#HiveServer2Clients-Beeline–CommandLineShell)からBeelineに関する情報を得ることができます。
* Beelineは[SQLLine](http://sqlline.sourceforge.net/)をベースにしています。

## STEP 2.3: EXPLORE HIVE SETTINGS ON AMBARI DASHBOARD

### 2.3.1 Open Ambari Dashboard in New Tab

DashboardタブをクリックしてAmbari DashboardからHiveの設定のページを探します。

![Dashboard ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/ambari_dashboard_lab2.png)

### 2.3.2 Become Familiar with Hive Settings

`Hive page`へ行き、`Configs tab`を選択し、`Settings tab`をクリックします。

![Hive Settings ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/ambari_dashboard_explanation_lab2.png)

Hive pageをクリックすると、上記と同様のページが表示されます。

1. Hive Page
2. Hive Configs Tab
3. Hive Settings Tab
4. Version History of Configuration

Optimization Settingsまでスクロールします。

![Optimization Settings ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/hive_optimization_lab2.png)

上のスクリーンショットでは、

1. Tezが最適化エンジンとして設定されている
2. コストベースオプティマイザ（CBO）が有効になっている

これは、設定を簡単にするHDP 2.5のAmbari Smart Configurationsです。

* Hadoopの設定は複数のXMLファイルで記述されます。
* 初期のバージョンのHadoopでは、オペレータは設定を変更するためにXML編集を行う必要がありました。デフォルトのバージョン管理はありませんでした。
* 初期のAmbariインターフェイスでは、さまざまな設定のダイアログボックスを使用して設定ページを表示し、それらを編集できるようにすることで、値を簡単に変更できました。しかし、そのフィールドの意味を調査して、値の範囲を理解する必要がありました。
* Smart Configurationsを使用すると、トグルスイッチで機能を切り替えたり、範囲を持つ設定ではスライダバーを使用したりできます。

デフォルトでは、重要な設定が最初のページに表示されます。探している設定がこのページにない場合は、Advancedタブで追加の設定を見つけることができます。

![Advanced Tab ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/hive_vectorization_lab2.png)

たとえば、SQLのパフォーマンスを向上させたい場合は、新しいHiveベクタライズ機能を使用できます。 これらの設定は、次の手順で見つけて有効にすることができます。

1. Advancedタブをクリックし、スクロールしてプロパティを見つけます。
2. あるいは、プロパティ検索フィールドにプロパティの入力すると、プロパティがフィルタされ見つけやすくなります。

上記の緑色のサークルからわかるように、`Enable Vectorization and Map Vectorization`は既にオンになっています。

vectorizationやその他のHiveのチューニングにあたって重要な設定の詳細は下記をご覧ください。

* Apache Hive docs on [Vectorized Query Execution](https://cwiki.apache.org/confluence/display/Hive/Vectorized+Query+Execution)
* [HDP Docs Vectorization docs](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/#section_4)
* [Hive Blogs](http://hortonworks.com/blog/category/hive/)
* [5 Ways to Make Your Hive Queries Run Faster](http://hortonworks.com/blog/5-ways-make-hive-queries-run-faster/)
* [Interactive Query for Hadoop with Apache Hive on Apache Tez](http://hortonworks.com/hadoop-tutorial/supercharging-interactive-queries-hive-tez/)
* [valuating Hive with Tez as a Fast Query Engine](http://hortonworks.com/blog/evaluating-hive-with-tez-as-a-fast-query-engine/)

## STEP 2.4: ANALYZE THE TRUCKS DATA

次に、Hive、Pig、およびZeppelinを使用して、geolocationテーブルとtrucksテーブルから派生データを分析します。ビズネスの目的は、運転者の疲労、過剰使用トラック、および様々な運送イベントがリスクに与える影響をより深く理解することです。これを達成するために、ソースデータにSQLを使用して一連の変換を適用し、PigまたはSparkを使用してリスクを計算します。 データ視覚化に関する最後のラボでは、Zeppelinを使用して一連のチャートを生成し、リスクをより深く理解します。

![Analyze the Tracks Data ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/Lab2_211.png)

最初の変換を始めましょう。各トラックの1ガロンあたりのマイル数を計算します。 trucksテーブルから始めます。私たちはトラックごとにmileカラムとgasカラムを合計する必要があります。Hiveには、テーブルを再フォーマットするために使用できる一連の関数があります。LATERAL VIEWについては、https://cwiki.apache.org/confluence/display/Hive/LanguageManual+LateralView をご覧ください。stack関数を用いることで、最大54行の、rdate、gas、mileという3つの列（例： 'june13'、june13_miles、june13_gas）にデータを再構成することができます。元のテーブルからtruckid、driverid、rdate、mile、gasを選択し、mpg（マイル/ガス）の計算列を追加します。そして、平均マイル数を計算します。

### 2.4.1 Create Table truck_mileage From Existing Trucking Data

Ambari Hive User Viewを使って、以下のクエリを実行します。

```
CREATE TABLE truck_mileage STORED AS ORC AS SELECT truckid, driverid, rdate, miles, gas, miles / gas mpg FROM trucks LATERAL VIEW stack(54, 'jun13',jun13_miles,jun13_gas,'may13',may13_miles,may13_gas,'apr13',apr13_miles,apr13_gas,'mar13',mar13_miles,mar13_gas,'feb13',feb13_miles,feb13_gas,'jan13',jan13_miles,jan13_gas,'dec12',dec12_miles,dec12_gas,'nov12',nov12_miles,nov12_gas,'oct12',oct12_miles,oct12_gas,'sep12',sep12_miles,sep12_gas,'aug12',aug12_miles,aug12_gas,'jul12',jul12_miles,jul12_gas,'jun12',jun12_miles,jun12_gas,'may12',may12_miles,may12_gas,'apr12',apr12_miles,apr12_gas,'mar12',mar12_miles,mar12_gas,'feb12',feb12_miles,feb12_gas,'jan12',jan12_miles,jan12_gas,'dec11',dec11_miles,dec11_gas,'nov11',nov11_miles,nov11_gas,'oct11',oct11_miles,oct11_gas,'sep11',sep11_miles,sep11_gas,'aug11',aug11_miles,aug11_gas,'jul11',jul11_miles,jul11_gas,'jun11',jun11_miles,jun11_gas,'may11',may11_miles,may11_gas,'apr11',apr11_miles,apr11_gas,'mar11',mar11_miles,mar11_gas,'feb11',feb11_miles,feb11_gas,'jan11',jan11_miles,jan11_gas,'dec10',dec10_miles,dec10_gas,'nov10',nov10_miles,nov10_gas,'oct10',oct10_miles,oct10_gas,'sep10',sep10_miles,sep10_gas,'aug10',aug10_miles,aug10_gas,'jul10',jul10_miles,jul10_gas,'jun10',jun10_miles,jun10_gas,'may10',may10_miles,may10_gas,'apr10',apr10_miles,apr10_gas,'mar10',mar10_miles,mar10_gas,'feb10',feb10_miles,feb10_gas,'jan10',jan10_miles,jan10_gas,'dec09',dec09_miles,dec09_gas,'nov09',nov09_miles,nov09_gas,'oct09',oct09_miles,oct09_gas,'sep09',sep09_miles,sep09_gas,'aug09',aug09_miles,aug09_gas,'jul09',jul09_miles,jul09_gas,'jun09',jun09_miles,jun09_gas,'may09',may09_miles,may09_gas,'apr09',apr09_miles,apr09_gas,'mar09',mar09_miles,mar09_gas,'feb09',feb09_miles,feb09_gas,'jan09',jan09_miles,jan09_gas) dummyalias AS rdate, miles, gas;
```

![Truckmileage ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/create_table_truckmileage_lab2.png)

### 2.4.2 Explore a sampling of the data in the truck_mileage table

上記のSQLによって生成されたデータを表示するには、truck_mileageの横にあるDatabase ExplorerでLoad Sample Dataアイコンをクリックします。そうすると、トラックとドライバーが行った各走行記録の一覧が表示されます。

![Select Truckmileage ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/select_data_truckmileage_lab2.png)

### 2.4.3 Use the Content Assist to build a query

1. SQLワークシートを作ります。

2. SELECT文の入力を開始しますが、最初の2文字のみを入力してください。
```
SE
```

3. Ctrl+spaceを押すと、次のポップアップダイアログウィンドウが表示されます。
![Lab2_24 ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/Lab2_24.png)
※通知コンテンツアシストでは、「SE」で始まるオプションがいくつか表示されます。これらのショートカットは、多くのカスタムクエリコードを書くときに便利です。

4. あなたの入力中にCtrl+spaceを使用して、次のクエリを入力して、どのようなコンテンツのアシストができるか、どのように動作するかを知ることができます。
```
SELECT truckid, avg(mpg) avgmpg FROM truck_mileage GROUP BY truckid;
```
![Lab2_28 ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/Lab2_28.png)

5. "Save as …"ボタンをクリックして、クエリを"average mpg"として保存します。
![Lab2_28 ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/Lab2_26.png)

6. Hive User Viewの上部にあるタブの1つである"Saved Queries"のリストにクエリが表示されます。

7. "average mpg"クエリを実行し、その結果を表示します。

### 2.4.4 Explore Explain Features of the Hive Query Editor

1. 次に、さまざまなExplain機能を調べて、クエリの実行をよりよく理解してみましょう.Text Explain、Visual Explain、およびTez Explainです。Explainボタンをクリックします。
![Lab2_28 ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/Lab2_27.png)

2. 以下のように、Tezジョブのフローが表示されます。
![tez_job_result_lab2 ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/tez_job_result_lab2.png)

3. Visual Explainを表示するには、右のタブのVisual Explainアイコンをクリックします。これは、Explainの結果を視覚的にわかりやすく表示する機能です。
![tez_job_result_lab2 ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/visual_explain_dag_lab2.png)

### 2.4.5 Explore TEZ

...省略...

### 2.4.6 Create Table truck avg_mileage From Existing trucks_mileage Data

これらの結果をテーブルに保存します。これはHiveのかなり一般的なパターンで、[Create Table As Select](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-CreateTableAsSelect(CTAS))（CTAS）と呼ばれます。次のSQLを新しいワークシートに貼り付け、実行ボタンをクリックします。

```
CREATE TABLE avg_mileage
STORED AS ORC
AS
SELECT truckid, avg(mpg) avgmpg
FROM truck_mileage
GROUP BY truckid;
```

![Create avg mileage table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/create_avg_mileage_table_lab2.png)

### 2.4.7 Load Sample Data of avg_mileage

上記のSQLによって生成されたデータを表示するには、avg_mileageの横にあるDatabase ExplorerでLoad Sample Dataアイコンをクリックします。

![Load sample avg mileage ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/load_sample_avg_mileage_lab2.png)

## STEP 2.5: DEFINE TABLE SCHEMA

これでトラックデータを洗練し、各トラックの平均mpgを得ました（avg_mileageテーブル）。次のタスクは、各ドライバーのリスクファクターを計算することです。これは合計走行マイル数/異常イベント数です。geolocationテーブルからイベント情報を取得できます。

![Geolocation table ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/geolocation_table_lab3.png)

truck_mileageテーブルを見ると、driveridと各走行記録のマイル数が表示されます。各ドライバーの合計マイル数を取得するには、それらのレコードをdriveridでグループ化し、マイルを合計します。

### 2.5.1 Create Table DriverMileage from Existing truck_mileage Data

まず、truck_mileageから必要な列のクエリから作成されたDriverMileageという名前のテーブルを作成します。次のクエリは、driveridによってレコードをグループ化し、selectステートメントのマイルを合計します。以下のクエリを新しいワークシートで実行します。

```
CREATE TABLE DriverMileage
STORED AS ORC
AS
SELECT driverid, sum(miles) totmiles
FROM truck_mileage
GROUP BY driverid;
```

![Create driver mileage ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/driver_mileage_table_lab3.png)

注：このテーブルは、Pig LatinとSparkの両方のジョブにとって必要です。

### 2.5.2 View Data Generated by Query

データを表示するには、DriverMileageの横のDatabase ExplorerでLoad sample dataアイコンをクリックします。 結果は次のようになります。

![Create driver mileage ](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/select_data_drivermileage_lab2.png)

### 2.5.3 Explore Hive Data Visualization

...省略...

## SUMMARY

おめでとうございます！ geolocationデータとtruckデータを処理、フィルタリング、操作するために学んだHiveコマンドをまとめます。
CREATE TABLEを使用してHiveテーブルを作成し、LOAD DATA INPATHコマンドを使用してデータをロードできます。さらに、テーブルのファイル形式をORCに変更する方法を学びました。ORCを用いることで、データの読み取り、書き込み、および処理の効率が向上します。SELECT {column_name ...} FROM {table_name}を使用して既存のテーブルからパラメータを取得し、新しいフィルタテーブルを作成する方法を学びました。

## SUGGESTED READINGS

さらにHiveについて学びたい方は下記のリソースをご覧ください。

* [Apache Hive](http://hortonworks.com/hadoop/hive/)
* [Hive LLAP enables sub second SQL on Hadoop](http://hortonworks.com/blog/llap-enables-sub-second-sql-hadoop/)
* [Programming Hive](https://www.amazon.com/dp/1449319335)
* [Hive Language Manual](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL)
* [HDP DEVELOPER: APACHE PIG AND HIVE](http://hortonworks.com/training/class/hadoop-2-data-analysis-pig-hive/)
