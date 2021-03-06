---
title: "Azure HDInsight で Apache Storm によって Azure Data Lake Store を使用する"
description: "HDInsight 上の Apache Storm トポロジから Azure Data Lake Store にデータを書き込む方法について説明します。 このドキュメントおよび関連する例では、Data Lake Store への書き込みに HdfsBolt コンポーネントを使用する方法をデモンストレーションします。"
services: hdinsight
documentationcenter: na
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: 1df98653-a6c8-4662-a8c6-5d288fc4f3a6
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/12/2017
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: 279990a67ae260b09d056fd84a12160150eb4539
ms.openlocfilehash: 0342c13e48d3f3605dcc169523d7d8d2d7aedba8
ms.lasthandoff: 01/18/2017


---
# <a name="use-azure-data-lake-store-with-apache-storm-with-hdinsight-java"></a>HDInsight で Apache Storm によって Azure Data Lake Store を使用する (Java)

Azure Data Lake Store は、データの高スループット、可用性、耐久性、および信頼性を提供する、HDFS 互換のクラウド ストレージ サービスです。 このドキュメントでは、Java ベースの Storm トポロジを使用し、[HdfsBolt](http://storm.apache.org/releases/1.0.2/javadocs/org/apache/storm/hdfs/bolt/HdfsBolt.html) コンポーネントでデータを Azure Data Lake Store に書き込む方法について説明します。このコンポーネントは、Apache Storm の一部として提供されます。

> [!IMPORTANT]
> このドキュメントで使用されるトポロジの例は、HDInsight クラスターの Storm に含まれているコンポーネントに依存しており、他の Apache Storm クラスターで使用する場合には Azure Data Lake Store を動作させるための修正が必要になることがあります。

## <a name="how-to-work-with-azure-data-lake-store"></a>Azure Data Lake Store の操作方法

Data Lake Store は、HDFS 互換ファイル システムとして HDInsight に表示されるため、Storm-HDFS ボルトを使用して書き込むことができます。 次のコードは、`MYDATALAKE` という名前の Data Lake Store アカウントで、Storm-HDFS ボルトを使用して、`/stormdata` という名前のディレクトリにデータを書き込む方法を示しています。

```java
// 1. Create sync and rotation policies to control when data is synched
//    (written) to the file system and when to roll over into a new file.
SyncPolicy syncPolicy = new CountSyncPolicy(1000);
FileRotationPolicy rotationPolicy = new FileSizeRotationPolicy(0.5f, Units.KB);
// 2. Set the format. In this case, comma delimited
RecordFormat recordFormat = new DelimitedRecordFormat().withFieldDelimiter(",");
// 3. Set the directory name. In this case, '/stormdata/'
FileNameFormat fileNameFormat = new DefaultFileNameFormat().withPath("/stormdata/");
// 4. Create the bolt using the previously created settings,
//    and also tell it the base URL to your Data Lake Store.
// NOTE! Replace 'MYDATALAKE' below with the name of your data lake store.
HdfsBolt adlsBolt = new HdfsBolt()
    .withFsUrl("adl://MYDATALAKE.azuredatalakestore.net/")
        .withRecordFormat(recordFormat)
        .withFileNameFormat(fileNameFormat)
        .withRotationPolicy(rotationPolicy)
        .withSyncPolicy(syncPolicy);
// 4. Give it a name and wire it up to the bolt it accepts data
//    from. NOTE: The name used here is also used as part of the
//    file name for the files written to Data Lake Store.
builder.setBolt("ADLStoreBolt", adlsBolt, 1)
    .globalGrouping("finalcount");
```

> [!IMPORTANT]
> HDInsight から Data Lake Store を使用する場合は、`adl://` URI スキームを使用してください。

## <a name="prerequisites"></a>前提条件

* [Java JDK 1.7](https://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html) 以上

* [Maven 3.x](https://maven.apache.org/download.cgi)

* HDInsight クラスター バージョン 3.2 の Storm。 HDInsight クラスターに新しい Storm を作成するには、 [Azure での HDInsight と Data Lake Store の使用](../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md) に関するドキュメントの手順に従います。 このドキュメントの手順では、新しい HDInsight クラスターと Azure Data Lake Store の作成手順を説明します。  
  
  > [!IMPORTANT]
  > HDInsight クラスターを作成するときに、クラスターの種類として **Storm** を選択し、バージョンとして **3.2** を選択する必要があります。 OS には、Windows または Linux を指定できます。

### <a name="configure-environment-variables"></a>環境変数を構成する

開発用ワークステーションに Java と JDK をインストールするときに、次のような環境変数が設定される場合があります。 ただし、これらが存在するかどうかや、システムに対して適切な値が含まれているかを確認する必要があります。

* **JAVA_HOME** - Java ランタイム環境 (JRE) がインストールされているディレクトリを指している必要があります。 たとえば、Unix や Linux ディストリビューションの場合は、 `/usr/lib/jvm/java-7-oracle`のような値になります。 Windows の場合は、 `c:\Program Files (x86)\Java\jre1.7`のような値になります。
* **PATH** - 次のパスを含む必要があります。
  
  * **JAVA\_HOME** または同等のパス
  * **JAVA\_HOME\bin** または同等のパス
  * Maven がインストールされているディレクトリ

## <a name="topology-implementation"></a>トポロジの実装

このドキュメントで使用される例は、Java で記述され、以下のコンポーネントを使用します。

* **TickSpout**: トポロジ内の他のコンポーネントによって使用されるデータを生成します。
* **PartialCount**: TickSpout によって生成されたイベントをカウントします。
* **FinalCount**: PartialCount のカウント データを集計します。
* **ADLStoreBolt**: [HdfsBolt](http://storm.apache.org/releases/1.0.2/javadocs/org/apache/storm/hdfs/bolt/HdfsBolt.html) コンポーネントを使用して Azure Data Lake Store にデータを書き込みます。

このトポロジを含むプロジェクトは、 [https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store](https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store)からダウンロードして利用できます。

### <a name="understanding-adlstorebolt"></a>ADLStoreBolt について

ADLStoreBolt は、Azure Data Lake への書き込みを行うトポロジで HdfsBolt インスタンスのために使用される名前です。 これは、Microsoft によって作成された特別なバージョンの HdfsBolt ではありません。しかし、core-site 構成値に依存しており、Data Lake と通信するために Azure HDInsight に含まれている Hadoop コンポーネントにも依存しています。

具体的には、HDInsight クラスターを作成するときに、それを Azure Data Lake Store と関連付けることができます。 そうすることで、選択した Data Lake Store の core-site にエントリが書き込まれます。これらのエントリは、Data Lake Store との通信を可能にするために、hadoop クライアントや hadoop hdfs などのコンポーネントによって使用されます。

> [!NOTE]
> Microsoft は、Azure Data Lake Store および Azure Blob Storage との通信を可能にするコードを Apache Hadoop および Storm プロジェクトに提供していますが、この機能は他の Hadoop および Storm の配布には既定では含まれていない場合があります。

トポロジ内の HdfsBolt の構成は、次のとおりです。

    // 1. Create sync and rotation policies to control when data is synched
    //    (written) to the file system and when to roll over into a new file.
    SyncPolicy syncPolicy = new CountSyncPolicy(1000);
    FileRotationPolicy rotationPolicy = new FileSizeRotationPolicy(0.5f, Units.KB);
    // 2. Set the format. In this case, comma delimited
    RecordFormat recordFormat = new DelimitedRecordFormat().withFieldDelimiter(",");
    // 3. Set the directory name. In this case, '/stormdata/'
    FileNameFormat fileNameFormat = new DefaultFileNameFormat().withPath("/stormdata/");
    // 4. Create the bolt using the previously created settings,
    //    and also tell it the base URL to your Data Lake Store.
    // NOTE! Replace 'MYDATALAKE' below with the name of your data lake store.
    HdfsBolt adlsBolt = new HdfsBolt()
        .withFsUrl("adl://MYDATALAKE.azuredatalakestore.net/")
          .withRecordFormat(recordFormat)
          .withFileNameFormat(fileNameFormat)
          .withRotationPolicy(rotationPolicy)
          .withSyncPolicy(syncPolicy);
    // 4. Give it a name and wire it up to the bolt it accepts data
    //    from. NOTE: The name used here is also used as part of the
    //    file name for the files written to Data Lake Store.
    builder.setBolt("ADLStoreBolt", adlsBolt, 1)
      .globalGrouping("finalcount");

HdfsBolt を使い慣れていればわかるように、URL 以外のすべてが標準的な構成です。 URL は、Azure Data Lake Store のルートへのパスです。

Data Lake Store への書き込みは、HdfsBolt を使用し、URL を変更するだけで済むため、HDFS または WASB への書き込みに HdfsBolt を使用する既存の任意のトポロジを利用することができ、Azure Data Lake Store を使用するための変更も簡単です。

## <a name="build-and-package-the-topology"></a>トポロジをビルドおよびパッケージ化する

1. サンプル プロジェクトを [https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store ](https://github.com/Azure-Samples/hdinsight-storm-azure-data-lake-store) から開発環境にダウンロードします。

2. エディターで `StormToDataLake\src\main\java\com\microsoft\example\StormToDataLakeStore.java` ファイルを開き、`.withFsUrl("adl://MYDATALAKE.azuredatalakestore.net/")` という記述がある行を見つけます。 **MYDATALAKE** を、HDInsight サーバーの作成時に使用した Azure Data Lake Store の名前に変更します。

3. コマンド プロンプト、ターミナル、またはシェル セッションで、ダウンロードしたプロジェクトのルート ディレクトリに移動し、次のコマンドを実行してトポロジをビルドおよびパッケージ化します。
   
        mvn compile
        mvn package
   
    ビルドとパッケージ化が完了すると、`target` という名前の新しいディレクトリが作成されます。このディレクトリには、`StormToDataLakeStore-1.0-SNAPSHOT.jar` という名前のファイルが含まれています。 このファイルに、コンパイルされたトポロジが含まれています。

## <a name="deploy-and-run-on-linux-based-hdinsight"></a>Linux ベースの HDInsight でデプロイおよび実行する

HDInsight クラスターで Linux ベースの Storm を作成した場合は、次の手順に従って、トポロジをデプロイおよび実行します。

1. 次のコマンドを使用して、トポロジを HDInsight クラスターにコピーします。 **USER** を、クラスターの作成時に使用した SSH ユーザー名に置き換えます。 **CLUSTERNAME** は、クラスターの名前に置き換えます。
   
        scp target\StormToDataLakeStore-1.0-SNAPSHOT.jar USER@CLUSTERNAME-ssh.azurehdinsight.net:StormToDataLakeStore-1.0-SNAPSHOT.jar
   
    メッセージが表示されたら、クラスターの SSH ユーザーの作成時に使用したパスワードを入力します。 パスワードではなく公開キーを使用している場合は、`-i` パラメーターを使用して、対応する秘密キーのパスを指定することが必要な場合があります。
   
   > [!NOTE]
   > 開発に Windows クライアントを使用している場合、`scp` コマンドがないことがあります。 その場合は、`pscp` を使用できます。このコマンドは、[http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) で入手できます。

2. アップロードが完了したら、次のようにして、SSH を使用して HDInsight クラスターに接続します。 **USER** を、クラスターの作成時に使用した SSH ユーザー名に置き換えます。 **CLUSTERNAME** は、クラスターの名前に置き換えます。
   
        ssh USER@CLUSTERNAME-ssh.azurehdinsight.net
   
    メッセージが表示されたら、クラスターの SSH ユーザーの作成時に使用したパスワードを入力します。 パスワードではなく公開キーを使用している場合は、 `-i` パラメーターを使用して、対応する秘密キーのパスを指定することが必要な場合があります。
   
   > [!NOTE]
   > 開発に Windows クライアントを使用している場合は、クラスターへの接続に PuTTY クライアントを使用するための情報について、 [SSH による Windows から Linux ベースの HDInsight への接続](hdinsight-hadoop-linux-use-ssh-windows.md) に関するページをご覧ください。

3. 接続したら、次のコマンドを使用してトポロジを開始します。
   
        storm jar StormToDataLakeStore-1.0-SNAPSHOT.jar com.microsoft.example.StormToDataLakeStore datalakewriter
   
    `datalakewriter`のフレンドリ名でトポロジが開始されます。

## <a name="deploy-and-run-on-windows-based-hdinsight"></a>Windows ベースの HDInsight でデプロイおよび実行する

1. Web ブラウザーを開いて HTTPS://CLUSTERNAME.azurehdinsight.net に移動します。**CLUSTERNAME** は実際の HDInsight クラスターの名前です。 メッセージが表示されたら、管理者ユーザー名 (`admin`) と、クラスターの作成時にこのアカウントに使用したパスワードを指定します。

2. Storm ダッシュボードの **[Jar ファイル]** ドロップダウンで **[参照]** を選択し、`target` ディレクトリの StormToDataLakeStore-1.0-SNAPSHOT.jar ファイルを選択します。 フォームの他のエントリには、以下の値を使用します。
   
   * クラス名: com.microsoft.example.StormToDataLakeStore
   * 追加のパラメーター: datalakewriter
     
  ![image of storm dashboard](./media/hdinsight-storm-write-data-lake-store/submit.png)

3. **[送信]** ボタンをクリックして、トポロジをアップロードおよび開始します。 トポロジが開始されると、**[送信]** ボタンの下にある結果フィールドに、次のような情報が表示されます。
   
        Process exit code: 0
        Currently running topologies:
        Topology_name        Status     Num_tasks  Num_workers  Uptime_secs
        -------------------------------------------------------------------
        datalakewriter       ACTIVE     68         8            10        

## <a name="view-output-data"></a>出力データを表示する

データを表示するには、いくつかの方法があります。 このセクションでは、Azure ポータルと `hdfs` コマンドを使用して、データを表示します。

> [!NOTE]
> Azure Data Lake Store のいくつかのファイルにデータが同期されるように、出力データを確認する前にトポロジを数分間実行します。


* **[Azure Portal](https://portal.azure.com) から**: ポータルで、HDInsight で使用した Azure Data Lake Store を選択します。
  
  > [!NOTE]
  > Data Lake Store を Azure Portal ダッシュボードにピン留めしなかった場合は、左側の一覧の下部にある **[参照]**、**[Data Lake Store]** の順に選択し、ストアを探して選択することができます。
  
    Data Lake Store の上部にあるアイコンの中から、**[データ エクスプローラー]** を選択します。
  
    ![data explore icon](./media/hdinsight-storm-write-data-lake-store/dataexplorer.png)
  
    次に、**stormdata** フォルダーを選択します。 テキスト ファイルの一覧が表示されます。
  
    ![text files](./media/hdinsight-storm-write-data-lake-store/stormoutput.png)
  
    いずれかのファイルを選択して、その内容を表示します。

* **クラスターから**: SSH (Linux クラスター) またはリモート デスクトップ (Windows クラスター) を使用して HDInsight クラスターに接続している場合は、次の方法でデータを表示できます。 **DATALAKE** を Data Lake Store の名前に置き換えます。
  
        hdfs dfs -cat adl://DATALAKE.azuredatalakestore.net/stormdata/*.txt
  
    これで、ディレクトリに格納されているテキスト ファイルが連結され、次のような情報が表示されます。
  
        406000000
        407000000
        408000000
        409000000
        410000000
        411000000
        412000000
        413000000
        414000000
        415000000

## <a name="stop-the-topology"></a>トポロジを停止する

Storm トポロジは、停止されるか、クラスターが削除されるまで実行され続けます。 トポロジを停止するには、次の情報を使用します。

**Linux ベースの HDInsight の場合**:

クラスターへの SSH セッションで、次のコマンドを使用します。

    storm kill datalakewriter

**Windows ベースの HDInsight の場合**:

1. Storm ダッシュボード (https://CLUSTERNAME.azurehdinsight.net) で、ページの上部にある **[Storm UI]** リンクを選択します。

2. Storm UI が読み込まれたら、**[datalakewriter]** リンクを選択します。
   
    ![link to datalakewriter](./media/hdinsight-storm-write-data-lake-store/selecttopology.png)

3. **[トポロジのアクション]** セクションで **[強制終了]** を選択し、表示されるダイアログ ボックスで [OK] を選択します。
   
    ![topology actions](./media/hdinsight-storm-write-data-lake-store/topologyactions.png)

## <a name="delete-your-cluster"></a>クラスターを削除する

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="next-steps"></a>次のステップ

ここでは、Storm を使用して Azure Data Lake Store に書き込む方法を学習しました。他の [HDInsight 用 Storm サンプル](hdinsight-storm-example-topology.md)もご覧ください。


