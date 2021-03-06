---
title: "Azure HDInsight で Apache Storm を使ってみる | Microsoft Docs"
description: "Linux ベースの HDInsight での Apache Storm および Storm Starter サンプルを使用したビッグ データ分析の概要 Storm を使用してデータをリアルタイムに処理する方法について説明します。"
keywords: "Apache Storm, Apache Storm チュートリアル, ビッグ データの分析, Storm Starter"
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: d710dcac-35d1-4c27-a8d6-acaf8146b485
ms.service: hdinsight
ms.devlang: java
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 01/12/2017
ms.author: larryfr
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: cfaade8249a643b77f3d7fdf466eb5ba38143f18
ms.openlocfilehash: 4950dbe528290c7d839c97cc8770db4ae0ec08c6
ms.lasthandoff: 03/01/2017

---
#<a name="get-started-with-the-storm-starter-samples-for-big-data-analytics-on-linux-based-hdinsight"></a>Storm Starter サンプルを使用した Linux ベースの HDInsight でのビッグ データ分析の概要

Apache Storm は、データ ストリームの処理を目的とし、スケーラビリティとフォールト トレランスに優れた、分散型のリアルタイム計算システムです。 Azure HDInsight の Storm を使用して、Storm でリアルタイムで ビッグ データ分析を実行するクラウドベースの Storm クラスターを作成できます。

> [!IMPORTANT]
> Linux は、バージョン 3.4 以上の HDInsight で使用できる唯一のオペレーティング システムです。 詳細については、[Window での HDInsight の廃止](hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date)に関する記事を参照してください。

## <a name="prerequisites"></a>前提条件

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

* **Azure サブスクリプション**。 [Azure 無料試用版の取得](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)に関するページを参照してください。

* **SSH と SCP を熟知していること**。 HDInsight での SSH と SCP の使用方法の詳細については、次のドキュメントを参照してください。
  
    * [HDInsight の Linux ベースの Hadoop で Windows 10、Linux、Unix、または OS X の Bash から SSH を使用する](hdinsight-hadoop-linux-use-ssh-unix.md)
    
    * [HDInsight の Linux ベースの Hadoop で Windows から SSH (PuTTY) を使用する](hdinsight-hadoop-linux-use-ssh-windows.md)

### <a name="access-control-requirements"></a>アクセス制御の要件

[!INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## <a name="create-a-storm-cluster"></a>Storm クラスターを作成する

HDInsight で Storm クラスターを作成するには、次の手順に従います。

1. [Azure Portal](https://portal.azure.com) で、**[+ 新規]**、**[インテリジェンス + 分析]**、**[HDInsight]** の順に選択します。
   
    ![HDInsight クラスターの作成](./media/hdinsight-apache-storm-tutorial-get-started-linux/create-hdinsight.png)

2. **[基本]** ブレードで、次の情報を入力します。

    * **[クラスター名]**: HDInsight クラスターの名前。
    * **[サブスクリプション]**: 使用するサブスクリプションを選択します。
    * **[クラスター ログイン ユーザー名]** と **[クラスター ログイン パスワード]**: HTTPS 経由でクラスターにアクセスする場合のログイン。 これらの資格情報を使用して、Ambari Web UI や REST API などのサービスにアクセスします。
    * **[Secure Shell (SSH) username (Secure Shell (SSH) ユーザー名)]**: SSH 経由でクラスターにアクセスする際に使用されるログイン。 既定では、このパスワードは、クラスター ログイン パスワードと同じです。
    * **[リソース グループ]**: クラスターが作成されるリソース グループ。
    * **[場所]**: クラスターが作成される Azure リージョン。
   
    ![サブスクリプションを選択します。](./media/hdinsight-apache-storm-tutorial-get-started-linux/hdinsight-basic-configuration.png)

3. **[クラスターの種類]** を選択し、**[クラスターの構成]** ブレードで次の値を設定します。
   
    * **[クラスターの種類]**: Storm

    * **[オペレーティング システム]**: Linux

    * **[バージョン]**: Storm 1.0.1 (HDI 3.5)

    * **[クラスター レベル]**: Standard
     
    最後に、**[選択]** ボタンをクリックして設定を保存します。
     
    ![クラスターの種類の選択](./media/hdinsight-apache-storm-tutorial-get-started-linux/set-hdinsight-cluster-type.png)

4. クラスターの種類を選択したら、__[選択]__ ボタンを使用してクラスターの種類を設定します。 次に、__[次へ]__ ボタンを使用して、基本的な構成を完了します。

5. **[ストレージ]** ブレードで、ストレージ アカウントを選択または作成します。 このドキュメントの手順では、このブレードの他のフィールドを既定値のままにします。 __[次へ]__ ボタンを使用して、ストレージの構成を保存します。

    ![HDInsight のストレージ アカウント設定](./media/hdinsight-apache-storm-tutorial-get-started-linux/set-hdinsight-storage-account.png)

6. **[概要]** ブレードで、クラスターの構成を確認します。 間違った設定を変更するには、__[編集]__ リンクを使用します。 最後に、__[作成]__ ボタンを使用してクラスターを作成します。
   
    ![クラスター構成の概要](./media/hdinsight-apache-storm-tutorial-get-started-linux/hdinsight-configuration-summary.png)
   
    > [!NOTE]
    > クラスターの作成には最大で 20 分かかります。

## <a name="run-a-storm-starter-sample-on-hdinsight"></a>HDInsight での Storm Starter サンプルの実行

1. SSH を使用して HDInsight クラスターに接続します。
   
        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
   
    SSH ユーザー アカウントを保護するためにパスワードを使用している場合は、パスワードの入力を求められます。 公開キーを使用している場合、`-i` パラメーターを使用して、対応する秘密キーを指定することが必要な場合があります。 たとえば、「 `ssh -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ssh.azurehdinsight.net`」のように入力します。
   
    Linux ベースの HDInsight での SSH の使用方法の詳細については、次の記事をご覧ください。
   
    * [HDInsight の Linux ベースの Hadoop で Windows 10、Linux、Unix、または OS X の Bash から SSH を使用する](hdinsight-hadoop-linux-use-ssh-unix.md)

    * [HDInsight の Linux ベースの Hadoop で Windows から SSH (PuTTY) を使用する](hdinsight-hadoop-linux-use-ssh-windows.md)

2. 次のコマンドを実行してトポロジの例を開始します。
   
        storm jar /usr/hdp/current/storm-client/contrib/storm-starter/storm-starter-topologies-*.jar storm jar org.apache.storm.starter.WordCountTopology wordcount
   
    > [!NOTE]
    > 以前のバージョンの HDInsight では、トポロジのクラス名は `org.apache.storm.starter.WordCountTopology` ではなく `storm.starter.WordCountTopology` です。
   
    このコマンドにより、クラスター上で、"wordcount" というフレンドリ名の WordCount トポロジの例が開始されます。 文はランダムに生成され、文中の各単語の出現回数がカウントされます。
   
    > [!NOTE]
    > 独自のトポロジをクラスターに送信する場合、まずクラスターを含む jar ファイルをコピーしてから、`storm` コマンドを実行します。 `scp` コマンドを使用して、ファイルをコピーします。 たとえば、 `scp FILENAME.jar USERNAME@CLUSTERNAME-ssh.azurehdinsight.net:FILENAME.jar`
    > 
    > WordCount の例と他の Storm スターターの例は、 `/usr/hdp/current/storm-client/contrib/storm-starter/`のクラスターに既に含まれています。

Storm スターターの例のコードを確認したい場合、そのコードは [https://github.com/apache/storm/tree/1.0.x-branch/examples/storm-starter](https://github.com/apache/storm/tree/1.0.x-branch/examples/storm-starter) で見つかります。 このリンクは、HDInsight 3.5 に付属している Storm 1.0.x 用です。 Storm の他のバージョンについては、このページの上部にある __[Branch (分岐)]__ ボタンを使用して、別の Storm バージョンを選択してください。

## <a name="monitor-the-topology"></a>トポロジの監視

Storm UI には、トポロジの実行を操作する Web インターフェイスがあり、HDInsight クラスターに含まれています。

次の手順により、Storm UI を使用してトポロジを監視します。

1. Storm UI を表示するには、Web ブラウザーを開いて、https://CLUSTERNAME.azurehdinsight.net/stormui を参照します。 **CLUSTERNAME** をクラスターの名前に置き換えます。
    
    > [!NOTE]
    > ユーザー名とパスワードの入力が求められたら、クラスターの作成時に使用したクラスター管理者名 (admin) とパスワードを入力します。

2. **[トポロジの概要]** で、**[名前]** 列の **[wordcount]** エントリを選択します。 トポロジの情報が表示されます。
    
    ![Storm Starter WordCount トポロジの情報が含まれている Storm ダッシュボード。](./media/hdinsight-apache-storm-tutorial-get-started-linux/topology-summary.png)
    
    このページには、次の情報が表示されます。
    
    * **トポロジの統計** - 時間枠で整理された、トポロジのパフォーマンスに関する基本的な情報。
     
        > [!NOTE]
        > 特定の時間枠を選択すると、ページの他のセクションに表示される情報の時間枠に変更されます。

    * **スパウト** - 各スパウトによって返された最後のエラーを含む、スパウト関する基本的な情報。
    
    * **ボルト** - ボルトに関する基本的な情報。
    
    * **トポロジの構成** - トポロジの構成に関する詳細情報。
     
    このページには、トポロジで実行できるアクションも表示されます。
   
    * **アクティブ化** - アクティブ化が解除されたトポロジの処理を再開します。
    
    * **アクティブ化の解除** - 実行中のトポロジを一時停止します
    
    * **再調整** - トポロジの並列処理を調整します。 クラスターのノード数を変更した場合は、実行中のトポロジを再調整する必要があります。 再調整によって、並列処理が、クラスター内のノード数の増減に合わせて調整されます。 詳細については、「[Understanding the parallelism of a Storm topology (Storm トポロジの並列処理)](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)」をご覧ください。
    
    * **強制終了** - 指定したタイムアウト後に Storm トポロジを停止します。

3. このページで、**[スパウト]** または **[ボルト]** セクションからエントリを選択します。 選択したコンポーネントに関する情報が表示されます。
   
    ![選択したコンポーネントに関する情報が含まれている Storm ダッシュボード。](./media/hdinsight-apache-storm-tutorial-get-started-linux/component-summary.png)
   
    このページには次の情報が表示されます。
   
    * **スパウト / ボルトの統計** - 時間枠で整理された、コンポーネントのパフォーマンスに関する基本的な情報。
     
        > [!NOTE]
        > 特定の時間枠を選択すると、ページの他のセクションに表示される情報の時間枠に変更されます。
     
    * **入力の統計** (ボルトのみ) - ボルトによって使用されるデータを生成するコンポーネントに関する情報。
    
    * **出力の統計** - このボルトによって出力されるデータに関する情報。
    
    * **エグゼキュータ** - このコンポーネントのインスタンスに関する情報。
    
    * **エラー** - このコンポーネントで生成されたエラー。

4. スパウトまたはボルトの詳細を表示したら、**[エグゼキュータ]** セクションの **[ポート]** 列でエントリを選択して、コンポーネントの特定のインスタンスの詳細を表示します。
   
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: split default ["with"]
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: split default ["nature"]
        2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [snow]
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [snow, 747293]
        2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [white]
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [white, 747293]
        2015-01-27 14:18:02 b.s.d.executor [INFO] Processing received message source: split:21, stream: default, id: {}, [seven]
        2015-01-27 14:18:02 b.s.d.task [INFO] Emitting: count default [seven, 1493957]
   
    この例では、**seven** という単語が 1493957 回発生しました。 この数値は、このトポロジが開始されてから、この単語が発生した回数です。

## <a name="stop-the-topology"></a>トポロジを停止する

ワードカウント トポロジの **[トポロジの概要]** ページに戻り、**[トポロジのアクション]** セクションで **[強制終了]** ボタンを選択します。 メッセージが表示されたら、トポロジを停止するまでの待機秒数として「10」を入力します。 タイムアウト期間後は、ダッシュボードの **[Storm UI]** セクションにアクセスしても、トポロジは表示されません。

## <a name="delete-the-cluster"></a>クラスターを削除する

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="a-idnextanext-steps"></a><a id="next"></a>次のステップ

この Apache Storm チュートリアルでは、HDInsight での Storm の操作の基本ついて説明しました。 次は、 [Maven を使用して Java ベースのトポロジを開発](hdinsight-storm-develop-java-topology.md)する方法について説明します。

既に Java ベースのトポロジの開発経験があり、既存のトポロジを HDInsight にデプロイする方法をお探しの方は、 [HDInsight での Apache Storm トポロジのデプロイと管理](hdinsight-storm-deploy-monitor-topology-linux.md)に関するページを参照してください。

.NET 開発者は、Visual Studio を使用して C# トポロジまたは C#/Java ハイブリッド トポロジを作成できます。 詳細については、「[Hadoop Tools for Visual Studio を使用した HDInsight での Apache Storm の C# トポロジの開発](hdinsight-storm-develop-csharp-visual-studio-topology.md)」をご覧ください。

HDInsight 上の Storm で使用できるトポロジの例については、以下の例をご覧ください。

* [HDInsight 上の Storm に関するトポロジ例](hdinsight-storm-example-topology.md)

[apachestorm]: https://storm.incubator.apache.org
[stormdocs]: http://storm.incubator.apache.org/documentation/Documentation.html
[stormstarter]: https://github.com/apache/storm/tree/master/examples/storm-starter
[stormjavadocs]: https://storm.incubator.apache.org/apidocs/
[azureportal]: https://manage.windowsazure.com/
[hdinsight-provision]: hdinsight-provision-clusters.md
[preview-portal]: https://portal.azure.com/

