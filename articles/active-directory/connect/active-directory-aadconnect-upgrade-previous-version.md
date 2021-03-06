---
title: "Azure AD Connect: 旧バージョンからアップグレードする | Microsoft Docs"
description: "インプレース アップグレードやスウィング移行など、Azure Active Directory Connect を最新リリースにアップグレードするさまざまな方法について説明します。"
services: active-directory
documentationcenter: 
author: AndKjell
manager: femila
editor: 
ms.assetid: 31f084d8-2b89-478c-9079-76cf92e6618f
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: Identity
ms.date: 02/08/2017
ms.author: billmath
translationtype: Human Translation
ms.sourcegitcommit: 1f7ec5d53512dcfbff17269802c8889eae0ad744
ms.openlocfilehash: 5dd69a0b9357a601070765817a814dae3e7e5c05


---
# <a name="azure-ad-connect-upgrade-from-a-previous-version-to-the-latest"></a>Azure AD Connect: 旧バージョンから最新バージョンにアップグレードする
このトピックでは、Azure AD Connect のインストールを最新リリースにアップグレードするさまざまな方法について説明します。 Azure AD Connect を常に最新リリースにしておくことをお勧めします。 「[スウィング移行](#swing-migration)」で説明されている手順は、構成を大幅に変更するときにも使用します。

DirSync からアップグレードする場合は、代わりに [Azure AD 同期ツール (DirSync) からのアップグレード](active-directory-aadconnect-dirsync-upgrade-get-started.md)を参照してください。

Azure AD Connect のアップグレードには複数の方法があります。

| メソッド | Description |
| --- | --- |
| [自動アップグレード](active-directory-aadconnect-feature-automatic-upgrade.md) |高速インストールのユーザーの場合、これは最も簡単な方法です。 |
| [インプレース アップグレード](#in-place-upgrade) |サーバーが&1; 台だけの場合は、同じサーバーでインストールをインプレース アップグレードします。 |
| [スウィング移行](#swing-migration) |2 台のサーバーを用意し、一方に新しいリリースまたは構成を準備して、アクティブなサーバーを変更します。 |

必要なアクセス許可については、[アップグレードに必要なアクセス許可](active-directory-aadconnect-accounts-permissions.md#upgrade)を参照してください。

> [!NOTE]
> Azure AD に対する変更の同期を新しい Azure AD Connect サーバーが開始できるようにした後は、DirSync または Azure AD Sync を使用してロールバックしないでください。 Azure AD Connect から DirSync、Azure AD Sync などの従来のクライアントへのダウングレードはサポートされておらず、Azure AD のデータ損失などの問題につながる場合があります。 
> 
> 

## <a name="in-place-upgrade"></a>インプレース アップグレード
インプレース アップグレードは、Azure AD Sync または Azure AD Connect からの移動に使用できます。 DirSync または FIM + Azure AD コネクタのソリューションには使用できません。

この方法は、サーバーが 1 台でオブジェクトが約 100,000 未満の場合にお勧めします。 標準の同期規則に対する変更があった場合は、アップグレード後にフル インポートと完全同期が実行されます。 これにより、新しい構成がシステムのすべての既存のオブジェクトに適用されることが保証されます。 同期エンジンのスコープ内のオブジェクトの数によっては、数時間かかることがあります。 通常の差分同期スケジューラー (既定では 30 分ごと) は中断されますが、パスワード同期は続行されます。 インプレース アップグレードは週末に実行するよう検討してください。 新しい Azure AD Connect リリースで標準構成に変更がなかった場合は、フル インポートまたは完全同期ではなく通常の差分インポートまたは差分同期が開始されます。  
![インプレース アップグレード](./media/active-directory-aadconnect-upgrade-previous-version/inplaceupgrade.png)

標準の同期規則を変更した場合は、アップグレードの間に既定の構成に戻ります。 アップグレードの間に構成が維持されるようにするには、「 [Azure AD Connect Sync: 既定の構成を変更するためのベスト プラクティス](active-directory-aadconnectsync-best-practices-changing-default-configuration.md)」の説明に従って変更を行ってください。

## <a name="swing-migration"></a>スウィング移行
複雑なデプロイまたは非常に多くのオブジェクトがある場合、ライブ システムでのインプレース アップグレードは困難である場合があります。 ユーザーによっては数日かかることがあり、この間差分変更は処理されません。 この方法は、構成を大幅に変更し、クラウドにプッシュする前にテストしたい場合にも使用されます。

このようなシナリオでは、スウィング移行を使用することをお勧めします。 アクティブ サーバーが&1; 台とステージング サーバーが&1; 台、(少なくとも)&2; 台のサーバーが必要です。 アクティブ サーバー (次の図の青い実線) は、アクティブな運用負荷を処理します。 ステージング サーバー (次の図の紫の破線) は新しいリリースまたは構成の準備用であり、このサーバーの準備が完全にできたらアクティブにします。 古いバージョンまたは構成がインストールされている前のアクティブ サーバーは、ステージング サーバーにしてアップグレードします。

2 つのサーバーには、それぞれ異なるバージョンを使用できます。 たとえば、使用を停止する予定のアクティブ サーバーでは Azure AD Sync を使用し、新しいステージング サーバーでは Azure AD Connect を使用することができます。 スウィング移行を使用して新しい構成を開発する場合は、2 つのサーバーで同じバージョンを使用することをお勧めします。  
![ステージング サーバー](./media/active-directory-aadconnect-upgrade-previous-version/stagingserver1.png)

注: このシナリオでは、3 台または&4; 台のサーバーを使用することが好まれる場合があります。 ステージング サーバーがアップグレードされているときに、[障害復旧](active-directory-aadconnectsync-operations.md#disaster-recovery)用のバックアップ サーバーがないためです。 3 台か&4; 台のサーバーを使用すると、新しいバージョンにアップグレードされたプライマリ/スタンバイ サーバーのセットを準備でき、引き継ぎ用のステージング サーバーを常に確保できます。

以下の手順は、Azure AD Sync または FIM + Azure AD Connector のソリューションからの移行にも使用できます。 この手順は DirSync には使用できませんが、DirSync 用の手順が組み込まれた同じスウィング移行 (並列デプロイとも呼ばれます) 方法が、[Azure Active Directory 同期 (DirSync) のアップグレード](active-directory-aadconnect-dirsync-upgrade-get-started.md)に関するページで説明されています。

### <a name="swing-migration-steps"></a>スウィング移行の手順
1. 両方のサーバーで Azure AD Connect を使用し、構成の変更だけを行う予定の場合は、アクティブ サーバーとステージング サーバーがどちらも同じバージョンを使用していることを確認します。 同じバージョンを使用することで、後で違いを比較しやすくなります。 Azure AD Sync からアップグレードする場合、2 つのサーバーのバージョンは異なるものになります。 Azure AD Connect の古いバージョンからアップグレードする場合は、2 つのサーバーで同じバージョンを使用している状態から開始することをお勧めしますが、必須ではありません。
2. カスタム構成を行っていて、ステージング サーバーにはそれが含まれない場合は、後の「 [カスタム構成をアクティブ サーバーからステージング サーバーに移動する](#move-custom-configuration-from-active-to-staging-server)」の手順に従ってください。
3. Azure AD Connect の以前のリリースからアップグレードする場合は、ステージング サーバーを最新のバージョンにアップグレードします。 Azure AD Sync から移行する場合は、ステージング サーバーに Azure AD Connect をインストールします。
4. 同期エンジンがステージング サーバーで完全なインポートと完全な同期を実行するまで待ちます。
5. 「 **サーバーの構成の確認** 」の「 [確認](active-directory-aadconnectsync-operations.md#verify-the-configuration-of-a-server)」の手順を使用して、新しい構成で予期しない変更が発生しなかったことを確認します。 予期しないことがあった場合は、データに問題がなくなるまで、修正、インポートと同期の実行、確認を繰り返します。 これらの手順については、リンクされているトピックを参照してください。
6. ステージング サーバーをアクティブ サーバーに切り替えます。 これは、「 **サーバーの構成の確認** 」の最後の手順である「 [アクティブなサーバーの切り替え](active-directory-aadconnectsync-operations.md#verify-the-configuration-of-a-server)」にあたります。
7. Azure AD Connect をアップグレードする場合は、ステージング モードになっているサーバーを最新リリースにアップグレードします。 前と同じ手順に従って、データと構成をアップグレードします。 Azure AD Sync からアップグレードしている場合は、ここで、以前のサーバーの電源を切って、使用を停止できます。

### <a name="move-custom-configuration-from-active-to-staging-server"></a>カスタム構成をアクティブ サーバーからステージング サーバーに移動する
アクティブ サーバーの構成を変更してある場合、ステージング サーバーにも同じ変更を適用する必要があります。

作成したカスタム同期規則は、PowerShell で移動できます。 その他の変更は、移行はできないため、両方のシステムで同じように適用する必要があります。

両方のサーバーで同じように構成されていることを確認する必要がある項目は次のとおりです。

* 同じフォレストへの接続。
* ドメインと OU のすべてのフィルター処理。
* 同じオプション機能 (パスワード同期やパスワード ライトバックなど)。

**同期規則の移動**  
カスタム同期規則を移動するには次のようにします。

1. アクティブ サーバーで **同期規則エディター** を開きます。
2. カスタム規則を選択します。 **[エクスポート]** をクリックします。 メモ帳ウィンドウが表示されます。 一時ファイルを PS1 という拡張子で保存します。 そうすることで、PowerShell スクリプトになります。 ps1 ファイルをステージング サーバーにコピーします。  
   ![同期規則のエクスポート](./media/active-directory-aadconnect-upgrade-previous-version/exportrule.png)
3. コネクタの GUID は、ステージング サーバーでは異なるものになり、変更する必要があります。 GUID を取得するには、**同期規則エディター**を開始し、同じ接続先システムを表す既定の規則のいずれかを選択して、**[エクスポート]** をクリックします。 PS1 ファイルの GUID を、ステージング サーバーから取得した GUID に置き換えます。
4. PowerShell プロンプトで、PS1 ファイルを実行します。 これにより、ステージング サーバーにカスタム同期規則が作成されます。
5. 複数のカスタム規則がある場合は、すべてのカスタム規則について繰り返します。

## <a name="next-steps"></a>次のステップ
「 [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)」をご覧ください。




<!--HONumber=Jan17_HO4-->


