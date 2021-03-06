---
title: "Apache Storm チュートリアル: Storm 入門 |Microsoft Docs"
description: "HDInsight での  Apache Storm および Storm Starter サンプルを使用したビッグ データ分析の概要 Storm を使用してデータをリアルタイムに処理する方法について説明します。"
keywords: "Apache Storm, Apache Storm チュートリアル, ビッグ データの分析, Storm Starter"
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 6f93f7ff-0736-4708-80ef-4289dae532a9
ms.service: hdinsight
ms.devlang: java
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 02/28/2017
ms.author: larryfr
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: cfaade8249a643b77f3d7fdf466eb5ba38143f18
ms.openlocfilehash: e505d02895abd011661b3e4f66c7f4f7ea042358
ms.lasthandoff: 03/01/2017

---
# <a name="get-started-with-the-storm-starter-samples-for-big-data-analytics-on-hdinsight"></a>Storm Starter サンプルを使用した HDInsight でのビッグ データ分析の概要

Apache Storm は、データ ストリームの処理を目的とし、スケーラビリティとフォールト トレランスに優れた、分散型のリアルタイム計算システムです。 Microsoft Azure HDInsight の Storm を使用して、Storm でリアルタイムで ビッグ データ分析を実行するクラウドベースの Storm クラスターを作成できます。 

> [!IMPORTANT]
> この記事の手順では、Windows ベースの HDInsight クラスターを作成します。 Windows では、バージョン 3.4 より前の HDInsight のみを使用できます。 Linux は、バージョン 3.4 以上の HDInsight で使用できる唯一のオペレーティング システムです。 詳細については、[Window での HDInsight の廃止](hdinsight-component-versioning.md#hdi-version-32-and-33-nearing-deprecation-date)に関する記事を参照してください。
>
> HDInsight クラスターで Linux ベースの Storm を作成する手順については、「 [Apache Storm チュートリアル: Storm Starter サンプルを使用した HDInsight でのビッグ データ分析の概要](hdinsight-apache-storm-tutorial-get-started-linux.md)

## <a name="prerequisites"></a>前提条件
[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

Apache Storm チュートリアルを正常に完了するには、次の条件を満たす必要があります。

* **Azure サブスクリプション**。 [Azure 無料試用版の取得](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)に関するページを参照してください。

### <a name="access-control-requirements"></a>アクセス制御の要件
[!INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## <a name="create-a-storm-cluster"></a>Storm クラスターを作成する

HDInsight で Storm クラスターを作成するには、次の手順に従います。

1. [Azure Portal](https://portal.azure.com) で、**[+ 新規]**、**[インテリジェンス + 分析]**、**[HDInsight]** の順に選択します。
   
    ![HDInsight クラスターの作成](./media/hdinsight-apache-storm-tutorial-get-started/create-hdinsight.png)

2. **[基本]** ブレードで、次の情報を入力します。

    * **[クラスター名]**: HDInsight クラスターの名前。
    * **[サブスクリプション]**: 使用するサブスクリプションを選択します。
    * **[クラスター ログイン ユーザー名]** と **[クラスター ログイン パスワード]**: HTTPS 経由でクラスターにアクセスする場合のログイン。 これらの資格情報を使用して、Ambari Web UI や REST API などのサービスにアクセスします。
    * **Secure Shell (SSH) のユーザー名**: これらのフィールドは既定値のままにします。 Windows ベースの HDInsight クラスターには使用されません。
    * **[リソース グループ]**: クラスターが作成されるリソース グループ。
    * **[場所]**: クラスターが作成される Azure リージョン。
   
    ![サブスクリプションを選択します。](./media/hdinsight-apache-storm-tutorial-get-started/hdinsight-basic-configuration.png)

3. **[クラスターの種類]** を選択し、**[クラスターの構成]** ブレードで次の値を設定します。
   
    * **[クラスターの種類]**: Storm

    * **[オペレーティング システム]**: Windows

    * **[バージョン]**: Storm 0.10.0 (HDI 3.3)

        > [!NOTE]
        > Linux オペレーティング システムでは、HDInsight バージョン 3.4 以降のみ利用可能です。

    * **[クラスター レベル]**: Standard
     
    最後に、**[選択]** ボタンをクリックして設定を保存します。
     
    ![クラスターの種類の選択](./media/hdinsight-apache-storm-tutorial-get-started/set-hdinsight-cluster-type.png)

4. クラスターの種類を選択したら、__[選択]__ ボタンを使用してクラスターの種類を設定します。 次に、__[次へ]__ ボタンを使用して、基本的な構成を完了します。

5. **[ストレージ]** ブレードで、ストレージ アカウントを選択または作成します。 このドキュメントの手順では、このブレードの他のフィールドを既定値のままにします。 __[次へ]__ ボタンを使用して、ストレージの構成を保存します。

    ![HDInsight のストレージ アカウント設定](./media/hdinsight-apache-storm-tutorial-get-started/set-hdinsight-storage-account.png)

6. **[概要]** ブレードで、クラスターの構成を確認します。 間違った設定を変更するには、__[編集]__ リンクを使用します。 最後に、__[作成]__ ボタンを使用してクラスターを作成します。
   
    ![クラスター構成の概要](./media/hdinsight-apache-storm-tutorial-get-started/hdinsight-configuration-summary.png)
   
    > [!NOTE]
    > クラスターの作成には最大で 20 分かかります。

## <a name="run-a-storm-starter-sample-on-hdinsight"></a>HDInsight での Storm Starter サンプルの実行
この Apache Storm チュートリアルでは、GitHub で Storm Starter サンプルを使用したビッグ データ分析の概要について説明します。

HDInsight クラスターの各 Storm には、クラスターで Storm トポロジをアップロードし実行するために使用する Storm ダッシュボードが付属しています。 各クラスターには、Storm ダッシュボードから直接実行できるトポロジのサンプルも付属しています。

### <a id="connect"></a>ダッシュボードへの接続
ダッシュボードは、**clustername** がクラスターの名前である **https://&lt;clustername>.azurehdinsight.net//** にあります。 スタート画面でクラスターを選択し、ブレードの上部にある **[ダッシュボード]** リンクを選択することで、ダッシュボードへのリンクを見つけることもできます。

![Storm ダッシュボードのリンクを使用した Azure ポータル](./media/hdinsight-apache-storm-tutorial-get-started/dashboard.png)

> [!NOTE]
> ダッシュボードに接続すると、ユーザー名とパスワードを入力するように求められます。 これはクラスターの作成時に使用した管理者の名前 (**admin**) とパスワードです。
> 
> 

Storm ダッシュボードが読み込まれると、 **[トポロジの送信]** フォームが表示されます。

![Storm Starter トポロジを Storm ダッシュボードで送信します。](./media/hdinsight-apache-storm-tutorial-get-started/submit.png)

**[トポロジの送信]** フォームを使用して、Storm トポロジが含まれている .jar ファイルをアップロードし、実行できます。 また、これにはクラスターに付属しているいくつかの基本的なサンプルも含まれています。

### <a id="run"></a>GitHub の Storm Starter プロジェクトからワードカウント サンプルを実行する
クラスターに付属しているサンプルには、ワードカウント トポロジのいくつかのバリエーションが含まれます。 これらのサンプルには、センテンスをランダムに出力する**スパウト**と各センテンスを個別の単語に分割し、各単語が発生した回数をカウントする**ボルト**が含まれています。 これらのサンプルは、Apache Storm の一部である [Storm Starter サンプル](https://github.com/apache/storm/tree/master/examples/storm-starter)で提供されています。

次の手順に従って、Storm Starter サンプルを実行します。

1. **[Jar ファイル]** ドロップダウンから **[StormStarter - WordCount]** を選択します。 これで、このサンプル用のパラメーターが **[クラス名]** フィールドと **[追加のパラメーター]** フィールドに入力されます。
   
    ![Storm ダッシュボードで選択されている Storm Starter WordCount。](./media/hdinsight-apache-storm-tutorial-get-started/submit.png)
   
   * **クラス名** - トポロジを送信する .jar ファイル内のクラス。
   * **追加パラメーター** - トポロジに必要なすべてのパラメーター。 この例では、フィールドに、送信したトポロジのわかりやすい名前を指定しています。
2. **[送信]**をクリックします。 しばらくすると、 **[結果]** フィールドにジョブの送信に使用したコマンドとコマンドの結果が表示されます。 **[エラー]** フィールドには、トポロジの送信時に発生したエラーが表示されます。
   
    ![Storm Starter WordCount の送信ボタンと結果。](./media/hdinsight-apache-storm-tutorial-get-started/submit-results.png)
   
   > [!NOTE]
   > 結果はトポロジが完了したことを示しません - **Storm トポロジがいったん開始されると、停止するまで実行されます。** ワードカウント トポロジは、ランダムなセンテンスを生成し、停止されるまで各単語の出現回数を計算します。
   > 
   > 

### <a id="monitor"></a>トポロジの監視
Storm UI を使用してトポロジを監視できます。

1. Storm ダッシュボードの上部で、 **[Storm UI]** を選択します。 クラスターの情報と実行中のすべてのトポロジに関する概要情報が表示されます。
   
    ![Storm Starter WordCount トポロジの概要を示す Storm ダッシュボード。](./media/hdinsight-apache-storm-tutorial-get-started/stormui.png)
   
    上記のページで、トポロジがアクティブな状態だった時間、使用されているワーカー、エグゼキュータ、タスクの数を確認できます。
   
   > [!NOTE]
   > **[名前]** 列には、前に **[追加パラメーター]** フィールドで指定した名前が含まれています。
   > 
   > 
2. **[トポロジの概要]** で、**[名前]** 列の **[wordcount]** エントリを選択します。 これにより、トポロジの詳細が表示されます。
   
    ![Storm Starter WordCount トポロジの情報が含まれている Storm ダッシュボード。](./media/hdinsight-apache-storm-tutorial-get-started/topology-summary.png)
   
    このページには、次の情報が表示されます。
   
   * **トポロジの統計** - 時間枠で整理された、トポロジのパフォーマンスに関する基本的な情報。
     
     > [!NOTE]
     > 特定の時間枠を選択すると、ページの他のセクションに表示される情報の時間枠に変更されます。
     > 
     > 
   * **スパウト** - 各スパウトによって返された最後のエラーを含む、スパウト関する基本的な情報。
   * **ボルト** - ボルトに関する基本的な情報。
   * **トポロジの構成** - トポロジの構成に関する詳細情報。
     
     このページには、トポロジで実行できるアクションも表示されます。
   * **アクティブ化** - アクティブ化が解除されたトポロジの処理を再開します。
   * **アクティブ化の解除** - 実行中のトポロジを一時停止します
   * **再調整** - トポロジの並列処理を調整します。 クラスターのノード数を変更した場合は、実行中のトポロジを再調整する必要があります。 この操作で、クラスター内のノード数の増減に合わせて、トポロジの並列処理を調整できます。 詳細については、「 [Understanding the parallelism of a Storm topology (Storm トポロジの並列処理)](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)」を参照してください。
   * **強制終了** - 指定したタイムアウト後に Storm トポロジを停止します。
3. このページで、**[スパウト]** または **[ボルト]** セクションからエントリを選択します。 選択したコンポーネントに関する情報が表示されます。
   
    ![選択したコンポーネントに関する情報が含まれている Storm ダッシュボード。](./media/hdinsight-apache-storm-tutorial-get-started/component-summary.png)
   
    このページには次の情報が表示されます。
   
   * **スパウト / ボルトの統計** - 時間枠で整理された、コンポーネントのパフォーマンスに関する基本的な情報。
     
     > [!NOTE]
     > 特定の時間枠を選択すると、ページの他のセクションに表示される情報の時間枠に変更されます。
     > 
     > 
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
   
    このデータから、 **seven** という単語が 1,493,957 回発生したことがわかります。 これは、このトポロジが開始されてから発生した回数です。

### <a name="stop-the-topology"></a>トポロジを停止する
ワードカウント トポロジの **[トポロジの概要]** ページに戻り、**[トポロジのアクション]** セクションで **[強制終了]** を選択します。 メッセージが表示されたら、トポロジを停止するまでの待機秒数として「10」を入力します。 タイムアウト期間後は、ダッシュボードの **[Storm UI]** セクションにアクセスしても、トポロジは表示されません。

## <a name="delete-the-cluster"></a>クラスターを削除する
[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="summary"></a>概要
この Apache Storm チュートリアルでは、Storm Starter を使用して、HDInsight クラスターで Storm を作成する方法と、Storm ダッシュボードを使用して Storm トポロジをデプロイ、監視、管理する方法について説明しました。

## <a id="next"></a>次のステップ
* **HDInsight Tools for Visual Studio** - HDInsight ツールでは、前述の Storm ダッシュボードのように、Visual Studio を使用して Storm トポロジを送信、監視、管理できます。 また HDInsight ツールは、C# Storm のトポロジを作成する機能を提供しており、クラスター上にデプロイし、実行できるトポロジ サンプルを含んでいます。
  
    詳細については、「 [HDInsight Tools for Visual Studio を使用して Hive クエリを実行する](hdinsight-hadoop-visual-studio-tools-get-started.md)」を参照してください。
* **サンプル ファイル** - HDInsight Storm クラスターでは、**%STORM_HOME%\contrib** ディレクトリにいくつかの例が用意されています。 次の例がそれぞれ含まれます。
  
  * ソース コード - storm-starter-0.9.1.2.1.5.0-2057-sources.jar など
  * Java ドキュメント - storm-starter-0.9.1.2.1.5.0-2057-javadoc.jar など
  * 例 - storm-starter-0.9.1.2.1.5.0-2057-jar-with-dependencies.jar など
    
    'jar' コマンドを使用して、ソース コードまたは Java ドキュメントを抽出します。 たとえば、'jar -xvf storm-starter-0.9.1.2.1.5.0.2057-javadoc.jar' のように指定します。
    
    > [!NOTE]
    > Java ドキュメントは Web ページで構成されます。 抽出後、ブラウザーを使用して **index.html** ファイルを表示します。
    > 
    > 
    
    これらのサンプルにアクセスするには、HDInsight クラスターの Storm のリモート デスクトップを有効にし、**%STORM_HOME%\contrib** からファイルをコピーする必要があります。
* 次のドキュメントには、HDInsight の Storm と使用できるその他のサンプルの一覧が含まれています。
  
  * [HDInsight 上の Storm に関するトポロジ例](hdinsight-storm-example-topology.md)

[apachestorm]: https://storm.incubator.apache.org
[stormdocs]: http://storm.incubator.apache.org/documentation/Documentation.html
[stormstarter]: https://github.com/apache/storm/tree/master/examples/storm-starter
[stormjavadocs]: https://storm.incubator.apache.org/apidocs/
[azureportal]: https://manage.windowsazure.com/
[hdinsight-provision]: hdinsight-provision-clusters.md
[preview-portal]: https://portal.azure.com/

