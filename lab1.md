
[LAB 1 - LOADING SENSOR DATA INTO HDFS](http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/#section_3)の翻訳です。

# LAB 1 - LOADING SENSOR DATA INTO HDFS

## INTRODUCTION

このセクションでは、センサデータをダウンロードし、Ambari User Viewを使用してHDFSにロードします。ファイルを管理するためにAmbari File User Viewを紹介します。Ambari File User Viewを用いるとディレクトリの作成、ファイルシステムのナビゲート、ファイルのHDFSへのアップロードなどのタスクを実行できます。さらに、他のファイル関連のタスクも実行することができます。最初にAmbari File User Viewの基礎を学習し、実際にAmbari File User Viewを用いて2つのディレクトリを作成し、2つのファイルをHDFSにロードしていきます。

## PRE-REQUISITES

このチュートリアルは、Hortonworks Sandboxを使用してHDPを始めるためのシリーズの一部です。このチュートリアルを進める前に、前提条件を満たしていることを確認してください。

* Hortonworks Sandboxをダウンロードしインストールしておく
* [Learning the Ropes of the Hortonworks Sandbox](http://hortonworks.com/hadoop-tutorial/learning-the-ropes-of-the-hortonworks-sandbox/) を読んでおく(オプション)

このチュートリアルを完了するのに20分ほどを要します。

## OUTLINE

* HDFS backdrop
* Step 1.1: Download data – Geolocation.zip
* Step 1.2: Load Data into HDFS
* Summary
* Suggested Reading

## HDFS BACKDROP

1台の物理マシンでは、データが増加するにつれてそのストレージ容量が限界に達してしまいます。このデータ増加のために、データを別々のマシンに分割する必要があります。ネットワークのマシン間でデータのストレージを管理するこのタイプのファイルシステムは、分散ファイルシステムと呼ばれます。HDFSはApache Hadoopの中心的なコンポーネントであり、コモディティハードウェアのクラスタ上で実行されるストリーミングデータアクセスパターンを含む大きなファイルを格納するように設計されています。Hortonworks Data Platform HDP 2.2では、HDFSはクラスタ内で[heterogeneous storage](http://hortonworks.com/blog/heterogeneous-storage-policies-hdp-2-2/)(様々なストレージメディア)をサポートするように拡張されました。

## STEP 1.1: DOWNLOAD AND EXTRACT THE SENSOR DATA FILES

1. サンプルセンサーデータはここからダウンロードできます：[Geolocation.zip](https://app.box.com/HadoopCrashCourseData)

2. Geolocation.zipをダウンロードし、ファイルを展開します。 次のファイルを含むGeolocationフォルダが展開されます。
  * geolocation.csv - これはトラックから収集されたジオロケーションデータです。トラックの場所、日付、時刻、イベントの種類、速度などを示すレコードが含まれています。
  * trucks.csv - これはリレーショナルデータベースからエクスポートされたデータで、トラックモデル、運転手ID、トラックID、および集計されたマイレージ(燃費)に関する情報が含まれています。

## STEP 1.2: LOAD THE SENSOR DATA INTO HDFS

1. Ambariのダッシュボードに移動し、HDFS File Viewを開きます。ユーザー名ボタンの横にある9つの四角のボタンをクリックし、Files Viewを選択します。
![Files  View](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/files_view_lab1.png)

2. HDFSファイルシステムの一番上のルートから始まり、ログインしているユーザー（この場合はmaria_dev）がアクセス権を持っているすべてのファイルが表示されます。
![Files  View](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/root_files_view_folder_lab1.png)

3. `/user/maria_dev`のディレクトリリンクをクリックして移動します。

4. 今回のユースケースで使用するデータをアップロードするためのデータディレクトリを作成しましょう。 ![New Folder](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/new_folder_icon_lab1.png)ボタンをクリックして、`maria_dev`ディレクトリ内にデータディレクトリを作成します。そして、`data`ディレクトリに移動します。
![Add New Folder](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/add_new_folder_data_lab1.png)

### 1.2.1 UPLOAD GEOLOCATION AND TRUCKS CSV FILES TO DATA FOLDER

5. 新しく作成したディレクトリパス`/user/maria_dev/data`にまだ移動していない場合は、そのフォルダに移動します。次に、![Upload](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/upload_icon_lab1.png)ボタンをクリックして、geolocation.csvとtrucks.csvをアップロードします。

6. ファイルのアップロードウィンドウが表示され、雲の画像をクリックします。
![Upload file ](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/upload_file_lab1.png)

7. 別のウィンドウが表示され流ので、2つのcsvファイルをダウンロードしたディレクトリに移動します。1回に1つのファイルを選択し、Openを押してアップロードを完了します。両方のファイルがアップロードされるまで、このプロセスを繰り返します。
![File Navigator ](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/upload_file_window_lab1.png)
両方のファイルがHDFSにアップロードされ、Files ViewのUIに表示されます。
![File Views ](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/uploaded_files_lab1.png)
ここでファイルやフォルダに対して、次の操作を実行することもできます： 開く、名前変更、権限変更、削除、コピー、移動、ダウンロード、ファイル連結

### 1.2.2 SET WRITE PERMISSIONS TO WRITE TO DATA FOLDER

8. ディレクトリパス`/user/maria_dev`に含まれている`data`フォルダをクリックします。[Permissions]をクリックします。下記の画像のように、すべてのWriteボックスがチェックされていることを確認してください（背景が青色になる）。

![File Views ](https://raw.githubusercontent.com/hortonworks/tutorials/hdp-2.5/assets/hello-hdp/edit_permissions_lab1.png)

## SUMMARY

おめでとうございます！ このチュートリアルで得たスキルと知識を要約しましょう。Hadoop Distributed File System（HDFS）は、複数のマシン間でデータを管理するために構築されたものです。 そしてAmbariのHDFS Files Viewを使用することでHDFSにデータをアップロードすることができます。

## SUGGESTED READING

* [HDFS](http://hortonworks.com/apache/hdfs/)
* [Manage Files on HDFS with Command Line: Hands-on Tutorial](http://hortonworks.com/hadoop-tutorial/using-commandline-manage-files-hdfs/)
* [HDFS User Guide](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html)
* Build your HDFS Architecture Knowledge [HDFS Architecture Guide](https://hadoop.apache.org/docs/r1.0.4/hdfs_design.html)
* [HDP OPERATIONS: HADOOP ADMINISTRATION](http://hortonworks.com/training/class/hdp-operations-hadoop-administration-1/)
