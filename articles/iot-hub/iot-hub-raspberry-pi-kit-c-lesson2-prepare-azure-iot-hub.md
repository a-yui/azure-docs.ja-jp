---
title: "Azure IoT への Raspberry Pi (C) の接続 - レッスン 2: デバイスの登録 | Microsoft Docs"
description: "Azure CLI を使用してリソース グループと Azure IoT Hub を作成し、Pi を Azure IoT Hub に登録します。"
services: iot-hub
documentationcenter: 
author: shizn
manager: timtl
tags: 
keywords: "Raspberry Pi クラウド, Pi クラウド接続"
ms.assetid: 4bcfb071-b3ae-48cc-8ea5-7e7434732287
ms.service: iot-hub
ms.devlang: c
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/28/2016
ms.author: xshi
translationtype: Human Translation
ms.sourcegitcommit: 64e69df256404e98f6175f77357500b562d74318
ms.openlocfilehash: d7bc143b509e84d50f3ce0164d31ae51a26f75d4


---
# <a name="create-your-iot-hub-and-register-raspberry-pi-3"></a>IoT ハブの作成と Raspberry Pi 3 の登録
## <a name="what-you-will-do"></a>学習内容
* リソース グループを作成します。
* リソース グループに Azure IoT ハブを作成します。
* Azure コマンド ライン インターフェイス (Azure CLI) を使用して Raspberry Pi 3 を Azure IoT ハブに追加します。

Pi を IoT Hub に追加するのに Azure CLI を使用する場合、サービスによる Pi の認証に使われるキーはサービスによって生成されます。 問題が発生した場合は、[トラブルシューティングのページ](iot-hub-raspberry-pi-kit-c-troubleshooting.md)で解決策を探してください。

## <a name="what-you-will-learn"></a>学習内容
この記事では、次のことについて説明します。
* Azure CLI を使用して IoT Hub を作成する方法。
* IoT Hub の Pi のデバイス ID を作成する方法。

## <a name="what-you-need"></a>必要なもの
* Azure アカウント
* Azure CLI がインストールされた Mac または Windows コンピューター

## <a name="create-your-iot-hub"></a>IoT ハブの作成
Azure IoT Hub は、何百万台もの IoT 資産を接続、監視、管理するのに役立ちます。 IoT ハブを作成するには、次の手順を実行します。

1. 次のコマンドを実行して Azure アカウントにサインインします。

   ```bash
   az login
   ```

   サインインの成功後に、使用できるサブスクリプションがすべて一覧表示されます。

2. 次のコマンドを実行して、既定で使用するサブスクリプションを設定します。

   ```bash
   az account set --subscription {subscription id or name}
   ```

   `subscription ID or name` は、`az login` または `az account list` コマンドの出力内で確認できます。

3. 次のコマンドを実行して、プロバイダーを登録します。 リソース プロバイダーは、アプリケーションにリソースを提供するサービスです。 プロバイダーによって提供される Azure リソースをデプロイする前に、プロバイダーを登録しておく必要があります。

   ```bash
   az provider register -n "Microsoft.Devices"
   ```
4. 次のコマンドを実行して、米国西部のリージョンに iot-sample という名前のリソース グループを作成します。

   ```bash
   az group create --name iot-sample --location westus
   ```

   `westus` は、リソース グループを作成する場所です。 別の場所を使用する場合は、`az account list-locations -o table` を実行して、Azure でサポートされているすべての場所を確認します。
 
5. 次のコマンドを実行して、リソース グループ iot-sample に IoT ハブを作成します。

   ```bash
   az iot hub create --name {my hub name} --resource-group iot-sample
   ```

   既定では、IoT ハブは Free 価格レベルで作成されます。 詳細については、「[Azure IoT Hub の価格](https://azure.microsoft.com/pricing/details/iot-hub/)」を参照してください。

> [!NOTE]
> IoT ハブの名前は、グローバルに一意である必要があります。 Azure サブスクリプションで作成できる F1 エディションの Azure IoT Hub は&1; つのみです。

## <a name="register-pi-in-your-iot-hub"></a>IoT ハブへの Pi の登録
IoT ハブとの間でメッセージの交換を行う各デバイスは、一意の ID を使用して登録する必要があります。

次のコマンドを実行して、Pi をハブに登録します。

```bash
az iot device create --device-id myraspberrypi --hub {my hub name} --resource-group iot-sample
```

## <a name="summary"></a>概要
IoT ハブを作成し、デバイス ID を使用して IoT ハブに Pi を登録しました。 これで、Pi から IoT ハブにメッセージを送信する方法を学習する準備が整いました。

## <a name="next-steps"></a>次のステップ
[IoT Hub のメッセージを処理して格納するための、Azure Function App と Azure ストレージ アカウントを作成](iot-hub-raspberry-pi-kit-c-lesson3-deploy-resource-manager-template.md)します。




<!--HONumber=Jan17_HO4-->


