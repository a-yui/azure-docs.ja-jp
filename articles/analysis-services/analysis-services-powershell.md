---
title: "PowerShell で Azure Analysis Services を管理する | Microsoft Docs"
description: "PowerShell を使用した Azure Analysis Services 管理。"
services: analysis-services
documentationcenter: 
author: minewiskan
manager: erikre
editor: 
ms.assetid: 
ms.service: analysis-services
ms.workload: data-management
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/24/2017
ms.author: owend
translationtype: Human Translation
ms.sourcegitcommit: ef3f31c633eeba92f343e2126626bd029aebbf64
ms.openlocfilehash: 170657601a0ea6b0c0ebabfd34befdce290cebd8


---

# <a name="manage-azure-analysis-services-with-powershell"></a>PowerShell で Azure Analysis Services を管理する

この記事では、Azure Analysis Services サーバーおよびデータベース管理タスクを実行するときに使用する PowerShell コマンドレットについて説明します。 

サーバーの作成/削除、サーバー操作の中断/開始、サービス レベルの変更などのサーバー管理タスクでは、Azure Resource Manager (AzureRM) が使用されます。 ロール メンバーの追加や削除、処理、パーティション分割など、その他のデータベース管理タスクについては、SQL Server Analysis Services と同じ [SQLASCMDLETS](https://msdn.microsoft.com/library/hh758425.aspx) モジュールのコマンドレットが使用されます。

## <a name="permissions"></a>アクセス許可
ほとんどの PowerShell タスクでは、管理している Analysis Services サーバーに対する管理者権限が必要となります。 スケジュールされた PowerShell タスクは無人操作です。 スケジューラを実行するアカウントにも、Analysis Services サーバーに対する管理者権限が必要です。 

AzureRm コマンドレットを使用したサーバー操作の場合、自分のアカウントまたはスケジューラを実行するアカウントが、[Azure のロールベースのアクセス制御 (RBAC)](../active-directory/role-based-access-control-what-is.md) でリソースの所有者ロールに属していることも必要になります。 

## <a name="server-operations"></a>サーバーの操作 
Azure Analysis Services コマンドレットは、[AzureRM.AnalysisServices](https://www.powershellgallery.com/packages/AzureRM.AnalysisServices) コンポーネント モジュールに含まれます。 AzureRM コマンドレット モジュールをインストールする場合は、PowerShell ギャラリーの [Azure Resource Manager コマンドレット](https://docs.microsoft.com/powershell/resourcemanager/)を確認してください。

|コマンドレット|Description| 
|------------|-----------------| 
|Get-AzureRmAnalysisServicesServer|サーバー インスタンスの詳細を取得します。|  
|New-AzureRmAnalysisServicesServer|新しいサーバー インスタンスを作成します。|
|Remove-AzureRmAnalysisServicesServer|サーバー インスタンスを削除します。|  
|Suspend-AzureRmAnalysisServicesServe|サーバー インスタンスを中断します。| 
|Resume-AzureRmAnalysisServicesServer|サーバー インスタンスを再開します。|  
|Set-AzureRmAnalysisServicesServer|サーバー インスタンスを変更します。|   
|Test-AzureRmAnalysisServicesServer|サーバー インスタンスの存在をテストします。| 

## <a name="database-operations"></a>データベース操作
Azure Analysis Services のデータベース操作では、SQL Server Analysis Services と同じ [SQLASCMDLETS](https://msdn.microsoft.com/library/hh758425.aspx) モジュールが使用されます。 ただし、すべてのコマンドレットが、Azure Analysis Services プレビューでサポートされているわけではありません。 

SQLASCMDLETS モジュールには、タスク固有のデータベース管理コマンドレットと、Tabular Model Scripting Language (TMSL) クエリまたはスクリプトを受け入れる汎用 Invoke-ASCmd コマンドレットが用意されています。 以下の SQLASCMDLETS モジュールのコマンドレットは、Azure Analysis Services プレビューでサポートされます。
  
|コマンドレット|説明|
|------------|-----------------| 
|[Add-RoleMember](https://msdn.microsoft.com/library/hh510167.aspx)|データベース ロールにメンバーを追加します。| 
|[Remove-RoleMember](https://msdn.microsoft.com/library/hh510173.aspx)|データベース ロールからメンバーを削除します。|   
|[Invoke-ASCmd](https://msdn.microsoft.com/library/hh479579.aspx)|TMSL スクリプトを実行します。|
|[Invoke-ProcessASDatabase](https://msdn.microsoft.com/library/mt651773.aspx)|データベースを処理します。|  
|[Invoke-ProcessPartition](https://msdn.microsoft.com/library/hh510164.aspx)|パーティションを処理します。| 
|[Invoke-ProcessTable](https://msdn.microsoft.com/library/mt651774.aspx)|テーブルを処理します。|  
|[Merge-Partition](https://msdn.microsoft.com/library/hh479576.aspx)|パーティションをマージします。|  
  

## <a name="related-information"></a>関連情報
* [Analysis Services の PowerShell スクリプト](https://msdn.microsoft.com/library/hh213141.aspx)。
* [互換性レベル 1200 の表形式モデル プログラミング](https://msdn.microsoft.com/library/mt712541.aspx)


<!--HONumber=Jan17_HO5-->


