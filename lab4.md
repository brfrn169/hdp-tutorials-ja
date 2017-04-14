
[LAB 4 - Spark Risk Factor Analysis](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/#section_6)の翻訳です。

# LAB 4 - SPARK RISK FACTOR ANALYSIS

## INTRODUCTION

このチュートリアルではApache Sparkを紹介します。このラボの初期のセクションでは、HDFSにデータをロードし、Hiveを使用してデータを操作する方法を学習しました。トラックのセンサーデータを使用して、すべてのドライバーに関連するリスクをよりよく理解できるようになりました。このセクションでは、Apache sparkを使用してリスクを計算する方法を説明します。

## PRE-REQUISITES

* [Learning the Ropes of the Hortonworks Sandbox](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/ "Learning the Ropes of the Hortonworks Sandbox")
* Hortonworks Sandbox
* Lab 1: Load sensor data into HDFS
* Lab 2: Data Manipulation with Apache Hive

このチュートリアルを完了するのに1時間ほどを要します。

## OUTLINE

* Apache Spark Backdrop
* Apache Spark Basics
* Step 4.1: Configure Apache Spark services using Ambari
* Step 4.2: Create a Hive Context
* Step 4.3: Create RDD from Hive Context
* Step 4.4: RDD transformations and actions
* Step 4.5: Load and save data into Hive as ORC
* Appendix A: Run Spark in the Spark Interactive Shell
* Appendix B: Login to Zeppelin (Optional)
* Summary
* Suggested Readings

## BACKGROUND

MapReduceは役に立ちましたが、ジョブの実行に要する時間は時にはとても大きくなることがあります。また、MapReduceジョブは特定のユースケースに対してのみ機能します。より広範なユースケースに対応するコンピューティングフレームワークが必要です。

Apache Sparkは、高速で汎用性の高い、使いやすいコンピューティングプラットフォームとして設計されています。これはMapReduceモデルを拡張し、それを他のすべてのレベルに引き継ぎます。速度はメモリ内の計算から得られます。メモリ内で実行されるアプリケーションは、処理と応答がずっと高速になります。

## APACHE SPARK

Apache Sparkは、Scala、Java、Python、およびRのエレガントで表現力豊かな開発APIを備えた高速メモリ内データ処理エンジンであり、データワーカーに迅速な反復アクセスが必要な機械学習アルゴリズムを効率的に実行できます。Spark on YARNは、Hadoopや他のエンタープライズにおける	YARN対応ワークロードとの深い統合を可能にします。

SparkではMapReduce型ジョブやイテレーティブなアルゴリズムなどのバッチアプリケーションを実行できます。インタラクティブなクエリを実行し、アプリケーションでストリーミングデータを処理することもできます。 Sparkには、機械学習アルゴリズム、SQL、ストリーミング、グラフ処理などを簡単に使用できる多数のライブラリも用意されています。Sparkは、Hadoop YARNやApache Mesos上で動作し、独自のスケジューラを使用するスタンドアロンモードでも動作します。Sandboxには、Spark 1.6とSpark 2.0の両方が含まれています。

![Spark](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/Lab4_1.png)

それでは始めましょう！

## STEP 4.1: CONFIGURE SPARK SERVICES USING AMBARI

1, Ambari Dashboardに`maria_dev`としてログオンします。 サービス列の左下にあるSparkとZeppelinが動作していることを確認します。

注：これらのサービスが無効になっている場合は、これらのサービスを開始してください。

![Ambari](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/ambari_dashboard_lab4.png)

### FOR HDP 2.5 SANDBOX USERS ACTIVATE LIVY SERVER

Livy Serverは、最新のSandbox HDP Platformに追加された新機能で、Zeppelin NotebookからのSparkジョブを実行しながら、特別なセキュリティを追加します。このラボでは、HDP 2.5 Sandboxを使用するユーザーはLivyを使用できます。

2, Spark Livyサーバーが動作していることを確認します。

![Livy Server](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/verify_spark_livy_server_lab4.png)

3, ご覧のように、私たちのサーバーはダウンしています。ZeppelinでSparkジョブを実行する前に起動する必要があります。`Livy Server`をクリックし、次に`sandbox.hortonworks.com`をクリックします。ここで`livy server`までスクロールし、`Stopped`ボタンを押してサーバーを起動しましょう。確認ウィンドウの`OK`ボタンを押します。

![Start Livy Server](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/start_livy_server_lab4.png)

Livy Serverがスタートしました。

![Started Livy Server](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/livy_server_running_lab4.png)

4, Sparkサービスに戻り、Service Actions -> Turn Off Maintenance Modeをクリックします。

Ambariからログアウトします。

5, `sandbox.hortonworks.com:9995`でZeppelinにアクセスしてください

Zeppelinのウェルカムページが表示されます。

![Zeppelin](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/zeppelin_welcome_page_lab4.png)

オプションで、Sparkでコードを実行するためにSpark Shellにアクセスする方法を知りたければ、Appendix Aを参照してください。

6, Zeppelin Notebookを作成してください

左上のNotebookタブをクリックし、Create new noteを押します。 ノートブックの名前を`Compute Riskfactor with Spark`に設定します。 デフォルトでは、ノートブックはSpark Scala APIをロードします。

![Zeppelin Note book](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/create_new_notebook_hello_hdp_lab4.png)

![Zeppelin create new note](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/notebook_name_hello_hdp_lab4.png)

## STEP 4.2: CREATE A HIVECONTEXT

Hive統合を強化により、HDP 2.5ではSparkのORCファイルをサポートしています。これにより、SparkはORCファイルに格納されたデータを読み取ることができます。 Sparkは、ORCファイルのより効率的なカラムストレージと述語プッシュダウン機能を活用して、より高速なメモリ内処理を実現します。HiveContextは、Hiveに格納されているデータと統合するSpark SQL実行エンジンのインスタンスです。より基本的なSQLContextは、Hiveに依存しないSpark SQLサポートのサブセットを提供します。 HiveContextは、クラスパス上のhive-site.xmlからHiveの設定を読み込みます。

### Import sql libraries:

...前半省略...

`％spark`インタプリタまたは`％livy`インタプリタを用いてSparkのコードを実行することができます。違いは、livyにはよりセキュアであるということです。 Sparkジョブのデフォルトのインタプリタは％sparkです。

```
%spark
import org.apache.spark.sql.hive.orc._
import org.apache.spark.sql._
```

![Import SQL Libraries](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/import_sql_libraries_hello_hdp_lab4.png)

### Instantiate HiveContext

```
%spark
val hiveContext = new org.apache.spark.sql.hive.HiveContext(sc)
```
![Instantiate HiveContext](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/instantiate_hivecontext_hello_hdp_lab4.png)

* `sc`はSpark Contextの略です。SparkContextは、すべてのSparkのメインエントリポイントです。これを使用して、クラスタ上でRDDと共有変数を作成できます。Sparkシェルを起動すると、変数`sc`でSparkContextが自動的に初期化されます。

## STEP 4.3: CREATE A RDD FROM HIVECONTEXT

### What is a RDD?

Sparkの主要なコア抽象化は、Resilient Distributed DatasetまたはRDDと呼ばれます。これは、クラスタ全体で並列化された要素の分散コレクションです。言い換えると、RDDは、YARNクラスタの複数の物理ノードに分割され、分散され、並列に操作できるオブジェクトの不変な集合です。

RDDを作成する方法は3つあります。
* 既存のコレクションを並列化します。これは、データがすでにSpark内に存在し、並列で操作できることを意味します。
* データセットを参照してRDDを作成します。 このデータセットは、HDFS、Cassandra、HBaseなどのHadoopでサポートされているすべてのストレージソースから取得できます。
* 既存のRDDを変換して新しいRDDを作成してRDDを作成します。

チュートリアルの後半の2つの方法を使用します。

### RDD Transformations and Actions

通常、RDDは、共有ファイルシステム、HDFS、HBase、またはYARNクラスタ上のHadoop InputFormatを提供するデータソースからデータをロードすることによってインスタンス化されます。

RDDがインスタンス化されると、一連の操作を適用できます。すべての操作は、変換またはアクションの2つのタイプのいずれかに分類されます。

* 名前からわかるように、変換操作では、既存のRDDから新しいデータセットを作成し、処理DAGを構築します。処理DAGは、YARNクラスタ全体でパーティションデータセットに適用できます。変換操作は値を返しません。実際には、これらの変換ステートメントの定義中に評価されるものはありません。 Sparkは、これらのDAGを作成します。これは実行時にのみ評価されます。私たちはこれをlazyな評価と呼びます。
* 一方、アクション操作は、DAGを実行して値を返します。

### 4.3.1 View List of Tables in Hive Warehouse

Hiveウェアハウス内のテーブルのリストを表示するには、showコマンドを使用します。

```
%spark
hiveContext.sql("show tables").collect.foreach(println)
```
![Show](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/view_list_tables_hive_hello_hdp_lab4.png)

注：falseは、列にデータが必要かどうかを示します。

すでにチュートリアルで作成したgeolocationテーブルとDriverMileageテーブルは既にHiveメタストアにリストされており、直接クエリできます。

### 4.3.2 Query Tables To Build Spark RDD

geolocationとDriverMileageテーブルからSpark変数にデータをフェッチする簡単なselectクエリを実行します。この方法でSparkにデータを取得することで、テーブルスキーマをRDDにコピーすることもできます。

```
%spark
val geolocation_temp1 = hiveContext.sql("select * from geolocation")
```

![Geolocation](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/query_tables_build_spark_rdd_hello_hdp_lab4.png)

```
%spark
val drivermileage_temp1 = hiveContext.sql("select * from drivermileage")
```

![DriverMileage](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/drivermileage_spark_rdd_hello_hdp_lab4.png)

## 4.4 Querying Against a Table

### 4.4.1 Registering a Temporary Table

次に、一時テーブルを登録して、SQL構文を使用してそのテーブルに対してクエリを実行します。

```
%spark
geolocation_temp1.registerTempTable("geolocation_temp1")
drivermileage_temp1.registerTempTable("drivermileage_temp1")
```

![Temporary](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/name_rdd_hello_hdp_lab4.png)

次に、反復とフィルタ操作を実行します。まず、正常ではないイベントを持つドライバをフィルタ処理し、各ドライバの非正常イベントの数をカウントする必要があります。

```
%spark
val geolocation_temp2 = hiveContext.sql("SELECT driverid, count(driverid) occurance from geolocation_temp1 where event!='normal' group by driverid")
```

![Normal](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/filter_drivers_nonnormal_events_hello_hdp_lab4.png)

選択(select)操作はRDD変換であるため、何も返されません。

結果の表には、各ドライバーに関連付けられた正常でないイベントの合計がカウントされます。 このフィルタリングされたテーブルを一時テーブルとして登録し、後続のSQLクエリをそのテーブルに適用できるようにします。

```
%spark
geolocation_temp2.registerTempTable("geolocation_temp2")
```

![Temporary2](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/register_filtered_table_hello_hdp_lab4.png)

RDDでアクション操作を実行すると、結果を表示できます。

```
%spark
geolocation_temp2.take(10).foreach(println)
```

![Results](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/view_results_op_on_rdd_hello_hdp_lab4.png)

### 4.4.2 Perform join Operation

このセクションでは、ジョイン操作を実行します。geolocation_temp2テーブルには、ドライバーの詳細とそれぞれの正常でないイベントのカウントがあります。 drivermileage_temp1テーブルには、各ドライバーが移動した総マイル数の詳細が入っています。

共通のカラムで2つの表を結合します。この例ではdriveridです。

```
%spark
val joined = hiveContext.sql("select a.driverid,a.occurance,b.totmiles from geolocation_temp2 a,drivermileage_temp1 b where a.driverid=b.driverid")

```

![Join](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/join_op_column_hello_hdp_lab4.png)

得られたデータセットは、特定のドライバーの合計マイル数と非通常の合計イベントを表示します。 このフィルタリングされたテーブルを一時テーブルとして登録し、後続のSQLクエリをそのテーブルに適用できるようにします。

```
%spark
joined.registerTempTable("joined")
```

![Temporary3](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/register_joined_table_hello_hdp_lab4.png)

RDDでアクション操作を実行すると、結果を表示できます。

```
%spark
joined.take(10).foreach(println)
```

![Results](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/show_results_joined_table_hello_hdp_lab4.png)

### 4.4.3 Compute Driver Risk Factor

このセクションでは、ドライバーのリスクファクタをすべてのドライバーに関連付けます。運転者のリスクファクタは、総走行距離から通常ではないイベントの発生総数を除算することによって計算されます。

```
%spark
val risk_factor_spark=hiveContext.sql("select driverid, occurance, totmiles, totmiles/occurance riskfactor from joined")
```

![Risk Factor](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/calculate_riskfactor_hello_hdp_lab4.png)

ここで得られたデータセットは、合計マイル数と通常ではないイベントの合計と、ドライバーにとってのリスクファクタが含まれています。このフィルタリングされたテーブルを一時テーブルとして登録し、後続のSQLクエリをそのテーブルに適用できるようにします。

```
%spark
risk_factor_spark.registerTempTable("risk_factor_spark")
```

RDDでアクション操作を実行すると、結果を表示できます。

```
%spark
risk_factor_spark.take(10).foreach(println)
```

![Results](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/view_results_filtertable_hello_hdp_lab4.png)

## STEP 4.5: LOAD AND SAVE DATA INTO HIVE AS ORC

このセクションでは、Sparkを使用してORC（Optimized Row Columnar）形式でデータを格納します。ORCは、Hadoopワークロード用に設計された自己記述型認識カラムファイル形式です。これは、大規模なストリーミングの読み込みと、必要な行をすばやく見つけるための統合サポートに最適化されています。データをカラム形式で保存すると、読者は現在のクエリに必要な値だけを読み込み、解凍し、処理することができます。ORCファイルはタイプを認識しているため、ライターはそのタイプに最も適したエンコーディングを選択し、ファイルが永続化されるときに内部インデックスを作成します。

述語プッシュダウンでは、これらの索引を使用して、特定の問合せに対してファイルのどのストライプを読み取る必要があるかを判断し、行索引を使用して10,000行の特定の集合に検索を絞り込むことができます。ORCは、構造体、リスト、マップ、および共用体の複合型を含むHiveの完全な型セットをサポートしています。

### 4.5.1 Create an ORC table

テーブルを作成し、ORCとして格納します。次のSQL文の最後にorcを指定すると、HiveテーブルがORC形式で格納されます。

```
%spark
hiveContext.sql("create table finalresults( driverid String, occurance bigint,totmiles bigint,riskfactor double) stored as orc").toDF()
```
注：toDF()でDataFrameが作成されます。

![orc](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/create_orc_table_hello_hdp_lab4.png)

### 4.5.2 Convert data into ORC table

上記で作成したHiveテーブルにデータをロードする前に、データファイルをORCフォーマットに変換する必要があります。

Spark 1.4.1かそれより上の場合は、以下を実行します。
```
%spark
risk_factor_spark.write.format("orc").save("risk_factor_spark")
```

上記を実行した場合は、次の手順をスキップして4.5.3に移動してください。


Spark 1.3.1の場合は、以下を実行します。
```
%spark
risk_factor_spark.saveAsOrcFile("risk_factor_spark")
```

![convert orc](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/convert_orc_table_hello_hdp_lab4.png)

### 4.5.3 Load the data into Hive table using load data command

```
%spark
hiveContext.sql("load data inpath 'risk_factor_spark' into table finalresults")
```

![load data to finalresults](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/load_data_to_finalresults_hello_hdp_lab4.png)

### 4.5.4 Create the final table Riskfactor using CTAS

```
%spark
hiveContext.sql("create table riskfactor as select * from finalresults")
```

![create table riskfactor](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/create_table_riskfactor_spark.png)


### 4.5.5 Verify Data Successfully Populated Hive Table in Hive (Check 2)

selectクエリを実行して、テーブルが正常に格納されたことを確認します。 Ambari Hive User Viewで、作成したHiveテーブルにデータが格納されているかどうかを確認できます。

![verify](https://raw.githubusercontent.com/hortonworks/data-tutorials/3b77c994580ba8cdb78a2dfdde76bd0e1a90e546/tutorials/hdp/hdp-2.5/hadoop-tutorial-getting-started-with-hdp/assets/riskfactor_table_populated.png)

## FULL SPARK CODE REVIEW FOR LAB

TODO

## APPENDIX A: RUN SPARK CODE IN THE SPARK INTERACTIVE SHELL

TODO

## SUMMARY

おめでとうございます！ すべてのドライバーに関連するリスクファクタを計算するために取得したSparkコーディングスキルと知識をまとめます。Apache Sparkは、メモリ内のデータ処理エンジンであるため非常に効率的です。HiveContextを作成することで、HiveとSparkを統合する方法を学びました。私たちはHiveの既存のデータを使用してRDDを作成しました。既存のRDDから新しいデータセットを作成するためのRDD変換とアクションを実行することを学びました。これらの新しいデータセットには、フィルタリングされ、操作され、処理されたデータが含まれます。リスクファクタを計算した後、データをHiveにロードしてORCとして保存することを学びました。

## SUGGESTED READINGS

さらにSparkについて学びたい方は下記のリソースをご覧ください。

* [Apache Spark](http://hortonworks.com/apache/spark/)
* [Apache Spark Welcome](http://spark.apache.org/)
* [Spark Programming Guide](http://spark.apache.org/docs/latest/programming-guide.html#passing-functions-to-spark)
* [Learning Spark](https://www.amazon.com/dp/1449358624)
* [Advanced Analytics with Spark](https://www.amazon.com/dp/1491912766)
* [HDP DEVELOPER: APACHE SPARK USING SCALA](http://hortonworks.com/training/class/hdp-developer-apache-spark-using-scala/)
* [HDP DEVELOPER: APACHE SPARK USING PYTHON](http://hortonworks.com/training/class/hdp-developer-apache-spark-using-python/)
