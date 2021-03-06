---
title: "Azure CLI のサンプル スクリプト - DSC を使用した IIS がインストールされた Windows Server 2016 の作成 | Microsoft Docs"
description: "Azure CLI のサンプル スクリプト - DSC を使用した IIS がインストールされた Windows Server 2016 の作成"
services: virtual-machines-windows
documentationcenter: virtual-machines
author: rickstercdn
manager: timlt
editor: tysonn
tags: 
ms.assetid: 
ms.service: virtual-machines-Windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 02/23/2017
ms.author: rclaus
translationtype: Human Translation
ms.sourcegitcommit: 82d40c30c92f5da090e7ec4e2f25ead3908cc603
ms.openlocfilehash: ea0598251e979094c5841c56c85f7297aaf3d21d
ms.lasthandoff: 03/02/2017

---

# <a name="create-a-vm-with-iis-using-dsc"></a>DSC を使用して、IIS で VM を作成する

このスクリプトでは、仮想マシンを作成してから、Azure 仮想マシンの DSC カスタム スクリプト拡張機能を使用して IIS をインストールおよび構成します。 

このスクリプトを実行する前に、`az login` コマンドを使用して Azure との接続が作成されていることを確認してください。 また、スクリプトの冒頭にある $AdminPassword 変数を、一意かつパスワードの複雑性要件を満たしたものに変更してください。

このサンプルは、bash シェルに対応しています。 Azure CLI スクリプトを Windows クライアントで実行する方法については、[Windows での Azure CLI の実行](../virtual-machines-windows-cli-options.md)に関する記事を参照してください。

## <a name="sample-script"></a>サンプル スクリプト

[!code-azurecli[main](../../../cli_scripts/virtual-machine/create-windows-iis-using-dsc/create-windows-iis-using-dsc.sh "VM の簡易作成")]

## <a name="clean-up-deployment"></a>デプロイのクリーンアップ 

サンプル スクリプトの実行後、次のコマンドを使用すると、リソース グループ、VM、およびすべての関連リソースを削除できます。

```azurecli
az group delete --name myResourceGroup --yes
```

## <a name="script-explanation"></a>スクリプトの説明

このスクリプトでは、次のコマンドを使用して、リソース グループ、仮想マシン、およびすべての関連リソースを作成します。 表内の各コマンドは、それぞれのドキュメントにリンクされています。

| コマンド | メモ |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | すべてのリソースを格納するリソース グループを作成します。 |
| [az vm create](https://docs.microsoft.com/cli/azure/vm#create) | 仮想マシンを作成し、作成したマシンをネットワーク カード、仮想ネットワーク、サブネット、およびNSG に接続します。 このコマンドでは、使用する仮想マシン イメージと管理者の資格情報も指定します。  |
| [az vm extension set](https://docs.microsoft.com/cli/azure/vm#create) | IIS のインストール スクリプトを呼び出すカスタム スクリプト拡張機能を、仮想マシンに追加します。 |
| [az vm open-port](https://docs.microsoft.com/cli/azure/vm#open-port) | 受信トラフィックを許可するネットワーク セキュリティ グループ規則を作成します。 このサンプルでは、HTTP トラフィック用にポート 80 を開きます。 |
| [az group delete](https://docs.microsoft.com/cli/azure/vm/extension#set) | 入れ子になったリソースすべてを含むリソース グループを削除します。 |

## <a name="next-steps"></a>次のステップ

Azure CLI の詳細については、[Azure CLI のドキュメント](https://docs.microsoft.com/cli/azure/overview)のページをご覧ください。

その他の仮想マシン用の CLI サンプル スクリプトは、[Azure Windows VM のドキュメント](../virtual-machines-windows-cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)のページにあります。
