---
title: "Azure CLI のサンプル - App Service | Microsoft Docs"
description: "Azure CLI のサンプル - App Service"
services: app-service
documentationcenter: app-service
author: syntaxc4
manager: erikre
editor: ggailey777
tags: azure-service-management
ms.assetid: 53e6a15a-370a-48df-8618-c6737e26acec
ms.service: app-service
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: app-service
ms.date: 02/21/2017
ms.author: cfowler
translationtype: Human Translation
ms.sourcegitcommit: f2c95ec5cd32632bb5b9683f5273721f465aebdf
ms.openlocfilehash: 2d7d2154a2910c0bd7ff24c404c4eb05203b1453
ms.lasthandoff: 03/01/2017


---
# <a name="azure-app-service-cli-samples"></a>Azure App Service の CLI サンプル

次の表には、Azure CLI を使用して構築された Bash スクリプトへのリンクが含まれています。

| | |
|-|-|
|**アプリの作成**||
| [GitHub からのデプロイで Web アプリを作成する](./scripts/app-service-cli-deploy-github.md)| GitHub からコードを取得する Azure Web アプリを作成します。 |
| [GitHub からの継続的なデプロイで Web アプリを作成する](./scripts/app-service-cli-continuous-deployment-github.md)| GitHub から継続的にコードをデプロイする Azure Web アプリを作成します。 |
| [Visual Studio Team Services からの継続的なデプロイで Web アプリを作成する](./scripts/app-service-cli-continuous-deployment-vsts.md)| Visual Studio Team Services から継続的にコードをデプロイする Azure Web アプリを作成します。 |
| [Web アプリを作成してローカル Git リポジトリからコードをデプロイする](./scripts/app-service-cli-deploy-local-git.md) | Azure Web アプリを作成し、ローカル Git リポジトリからのコードのプッシュを構成します。 |
| [Web アプリを作成してステージング環境にコードをデプロイする](./scripts/app-service-cli-deploy-staging-environment.md) | コードの変更をステージングするためのデプロイ スロットを持つ Azure Web アプリを作成します。 |
| [Docker Hub から Docker コンテナーに ASP.NET Core Web アプリを作成する](./scripts/app-service-cli-linux-docker-aspnetcore.md)| Linux 上で Azure Web アプリを作成し、Docker Hub から Docker イメージを読み込みます。 |
| [Azure コンテナー レジストリから Docker コンテナーに ASP.NET Core Web アプリケーションを作成する](./scripts/app-service-cli-linux-acr-aspnetcore.md)| Linux 上で Azure Web アプリを作成し、Azure コンテナー レジストリから Docker イメージを読み込みます。 |
|**アプリケーションの構成**||
| [カスタム ドメインを Web アプリにマップする](./scripts/app-service-cli-configure-custom-domain.md)| Azure Web アプリを作成し、カスタム ドメイン名をマップします。 |
|**アプリのスケール**||
| [Web アプリを手動でスケールする](./scripts/app-service-cli-scale-manual.md) | Azure Web アプリを作成し、2 つのインスタンス間でスケールします。 |
| [高可用性アーキテクチャを使用して世界規模で Web アプリをスケールする](./scripts/app-service-cli-scale-high-availability.md) | 2 つの異なる地理的リージョンに&2; つの Azure Web アプリを作成し、Azure Traffic Manager を使用して、1 つのエンドポイントを介して利用できるようにします。 |
|**アプリのリソースへの接続**||
| [Web アプリを SQL Database に接続する](./scripts/app-service-cli-app-service-sql.md)| Azure Web アプリと SQL データベースを作成し、データベース接続文字列をアプリケーション設定に追加します。 |
| [Web アプリをストレージ アカウントに接続する](./scripts/app-service-cli-app-service-storage.md)| Azure Web アプリとストレージ アカウントを作成し、ストレージ接続文字列をアプリケーション設定に追加します。 |
|**アプリの監視**||
| [Web サーバー ログによる Web アプリの監視](./scripts/app-service-cli-monitor.md) | Azure Web アプリを作成し、ログ記録を有効にし、ログをローカル コンピューターにダウンロードします。 |
| | |

