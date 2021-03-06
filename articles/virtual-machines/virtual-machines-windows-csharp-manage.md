---
title: "Azure Resource Manager と C# を使用した VM の管理 | Microsoft Docs"
description: "Azure Resource Manager と C# を使用して、仮想マシンを管理します。"
services: virtual-machines-windows
documentationcenter: 
author: davidmu1
manager: timlt
editor: 
tags: azure-resource-manager
ms.assetid: 93898bad-b861-4359-9a4b-348e1d491822
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.devlang: na
ms.topic: article
ms.date: 09/27/2016
ms.author: davidmu
translationtype: Human Translation
ms.sourcegitcommit: 0782000e87bed0d881be5238c1b91f89a970682c
ms.openlocfilehash: 81706c6c95199b25a07589780d2a27fc0f8dffa7


---
# <a name="manage-azure-virtual-machines-using-azure-resource-manager-and-c"></a>Azure Resource Manager と C を使用した Azure 仮想マシンの管理# #
この記事にあるタスクでは、開始、停止、および更新など、仮想マシンを管理する方法について説明します。 この記事のタスクを完了するには、リソース グループに仮想マシンが存在している必要があります。

この記事のタスクを完了するには、以下が必要です。

* [Visual Studio](http://msdn.microsoft.com/library/dd831853.aspx)
* [認証トークン](../azure-resource-manager/resource-group-authenticate-service-principal.md)

## <a name="create-a-visual-studio-project-and-install-packages"></a>Visual Studio プロジェクトの作成とパッケージのインストール
NuGet パッケージを使用すると、この記事のタスクを完了するために必要なライブラリを簡単にインストールできます。 この記事でインストールする必要があるライブラリは、Azure Active Directory 認証ライブラリと Compute リソース プロバイダー ライブラリです。 Visual Studio でこれらのライブラリを入手するには、次の手順を実行します。

1. **[ファイル]** > **[新規]** > **[プロジェクト]** の順にクリックします。
2. **[テンプレート]** > の **[Visual C#]** で **[コンソール アプリケーション]** を選択し、プロジェクトの名前と場所を入力して、**[OK]** をクリックします。
3. ソリューション エクスプローラーでプロジェクト名を右クリックし、 **[NuGet パッケージの管理]**をクリックします。
4. 検索ボックスに「 *Active Directory* 」と入力し、Active Directory Authentication Library パッケージの **[インストール]** をクリックして、パッケージのインストール手順に従います。
5. ページの上部で、 **[リリース前のパッケージを含める]**を選択します。 検索ボックスに「 *Microsoft.Azure.Management.Compute* 」と入力し、Compute .NET ライブラリの **[インストール]** をクリックして、パッケージのインストール手順に従います。

これで、ライブラリを使用して仮想マシンを管理する準備が整いました。

## <a name="set-up-the-project"></a>プロジェクトのセットアップ
これでアプリケーションが作成され、ライブラリがインストールされました。次は、アプリケーションの情報を使用してトークンを作成します。 このトークンは、Azure Resource Manager への要求を認証するのに使用されます。

1. 作成したプロジェクトの Program.cs ファイルを開き、次の using ステートメントをファイルの先頭に追加します。
   
        using Microsoft.Azure;
        using Microsoft.IdentityModel.Clients.ActiveDirectory;
        using Microsoft.Azure.Management.Compute;
        using Microsoft.Azure.Management.Compute.Models;
        using Microsoft.Rest;
2. Program クラスの Main メソッドに変数を追加し、リソース グループの名前、仮想マシンの名前、およびサブスクリプション ID を指定します。
   
        var groupName = "resource group name";
        var vmName = "virtual machine name";  
        var subscriptionId = "subsciption id";
   
    サブスクリプション ID は Get-AzureRmSubscription を実行して確認できます。
3. 資格情報の作成に必要なトークンを取得するために、このメソッドを Program クラスに追加します。
   
        private static async Task<AuthenticationResult> GetAccessTokenAsync()
        {
          var cc = new ClientCredential("{client-id}", "{client-secret}");
          var context = new AuthenticationContext("https://login.windows.net/{tenant-id}");
          var token = await context.AcquireTokenAsync("https://management.azure.com/", cc);
          if (token == null)
          {
            throw new InvalidOperationException("Could not get the token");
          }
          return token;
        }
   
    {client-id} を Azure Active Directory アプリケーションの ID に、{client-secret} を AD アプリケーションのアクセス キーに、および {tenant-id} をサブスクリプションのテナントID に置き換えます。 テナント ID は Get-AzureRmSubscription を実行して確認できます。 アクセス キーは、Azure ポータルで確認できます。
4. 資格情報を作成するには、Program.cs の Main メソッドにこのコードを追加します。
   
        var token = GetAccessTokenAsync();
        var credential = new TokenCredentials(token.Result.AccessToken);
5. Program.cs ファイルを保存します。

## <a name="display-information-about-a-virtual-machine"></a>仮想マシンに関する情報の表示
1. 前に作成したプロジェクトの Program クラスに、次のメソッドを追加します。
   
        public static async void GetVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Getting information about the virtual machine...");
   
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var vmResult = await computeManagementClient.VirtualMachines.GetAsync(
            groupName, 
            vmName, 
            InstanceViewTypes.InstanceView);
   
          Console.WriteLine("hardwareProfile");
          Console.WriteLine("   vmSize: " + vmResult.HardwareProfile.VmSize);
   
          Console.WriteLine("\nstorageProfile");
          Console.WriteLine("  imageReference");
          Console.WriteLine("    publisher: " + vmResult.StorageProfile.ImageReference.Publisher);
          Console.WriteLine("    offer: " + vmResult.StorageProfile.ImageReference.Offer);
          Console.WriteLine("    sku: " + vmResult.StorageProfile.ImageReference.Sku);
          Console.WriteLine("    version: " + vmResult.StorageProfile.ImageReference.Version);
          Console.WriteLine("  osDisk");
          Console.WriteLine("    osType: " + vmResult.StorageProfile.OsDisk.OsType);
          Console.WriteLine("    name: " + vmResult.StorageProfile.OsDisk.Name);
          Console.WriteLine("    createOption: " + vmResult.StorageProfile.OsDisk.CreateOption);
          Console.WriteLine("    uri: " + vmResult.StorageProfile.OsDisk.Vhd.Uri);
          Console.WriteLine("    caching: " + vmResult.StorageProfile.OsDisk.Caching);
   
          Console.WriteLine("\nosProfile");
          Console.WriteLine("  computerName: " + vmResult.OsProfile.ComputerName);
          Console.WriteLine("  adminUsername: " + vmResult.OsProfile.AdminUsername);
          Console.WriteLine("  provisionVMAgent: " + vmResult.OsProfile.WindowsConfiguration.ProvisionVMAgent.Value);
          Console.WriteLine("  enableAutomaticUpdates: " + vmResult.OsProfile.WindowsConfiguration.EnableAutomaticUpdates.Value);
   
          Console.WriteLine("\nnetworkProfile");
          foreach (NetworkInterfaceReference nic in vmResult.NetworkProfile.NetworkInterfaces)
          {
            Console.WriteLine("  networkInterface id: " + nic.Id);
          }
   
          Console.WriteLine("\nvmAgent");
          Console.WriteLine("  vmAgentVersion" + vmResult.InstanceView.VmAgent.VmAgentVersion);
          Console.WriteLine("    statuses");
          foreach (InstanceViewStatus stat in vmResult.InstanceView.VmAgent.Statuses)
          {
            Console.WriteLine("    code: " + stat.Code);
            Console.WriteLine("    level: " + stat.Level);
            Console.WriteLine("    displayStatus: " + stat.DisplayStatus);
            Console.WriteLine("    message: " + stat.Message);
            Console.WriteLine("    time: " + stat.Time);
          }
   
          Console.WriteLine("\ndisks");
          foreach (DiskInstanceView idisk in vmResult.InstanceView.Disks)
          {
            Console.WriteLine("  name: " + idisk.Name);
            Console.WriteLine("  statuses");
            foreach (InstanceViewStatus istat in idisk.Statuses)
            {
              Console.WriteLine("    code: " + istat.Code);
              Console.WriteLine("    level: " + istat.Level);
              Console.WriteLine("    displayStatus: " + istat.DisplayStatus);
              Console.WriteLine("    time: " + istat.Time);
            }
          }
   
          Console.WriteLine("\nVM general status");
          Console.WriteLine("  provisioningStatus: " + vmResult.ProvisioningState);
          Console.WriteLine("  id: " + vmResult.Id);
          Console.WriteLine("  name: " + vmResult.Name);
          Console.WriteLine("  type: " + vmResult.Type);
          Console.WriteLine("  location: " + vmResult.Location);
          Console.WriteLine("\nVM instance status");
          foreach (InstanceViewStatus istat in vmResult.InstanceView.Statuses)
          {
            Console.WriteLine("\n  code: " + istat.Code);
            Console.WriteLine("  level: " + istat.Level);
            Console.WriteLine("  displayStatus: " + istat.DisplayStatus);
          }
   
        }
2. 追加したメソッドを呼び出すために、次のコードを Main メソッドに追加します。
   
        GetVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
3. Program.cs ファイルを保存します。
4. Visual Studio で **[開始]** をクリックし、サブスクリプションで使用するのと同じユーザー名とパスワードを使用して Azure AD にサインインします。
   
    このメソッドを実行すると、次の例のように表示されます。
   
        Getting information about the virtual machine...
        hardwareProfile
          vmSize: Standard_A0
   
        storageProfile
          imageReference
            publisher: MicrosoftWindowsServer
            offer: WindowsServer
            sku: 2012-R2-Datacenter
            version: latest
          osDisk
            osType: Windows
            name: myosdisk
            createOption: FromImage
            uri: http://store1.blob.core.windows.net/vhds/myosdisk.vhd
            caching: ReadWrite
   
          osProfile
            computerName: vm1
            adminUsername: account1
            provisionVMAgent: True
            enableAutomaticUpdates: True
   
          networkProfile
            networkInterface 
              id: /subscriptions/{subscription-id}
                /resourceGroups/rg1/providers/Microsoft.Network/networkInterfaces/nc1
   
          vmAgent
            vmAgentVersion2.7.1198.766
            statuses
            code: ProvisioningState/succeeded
            level: Info
            displayStatus: Ready
            message: GuestAgent is running and accepting new configurations.
            time: 4/13/2016 8:35:32 PM
   
          disks
            name: myosdisk
            statuses
              code: ProvisioningState/succeeded
              level: Info
              displayStatus: Provisioning succeeded
              time: 4/13/2016 8:04:36 PM
   
          VM general status
            provisioningStatus: Succeeded
            id: /subscriptions/{subscription-id}
              /resourceGroups/rg1/providers/Microsoft.Compute/virtualMachines/vm1
            name: vm1
            type: Microsoft.Compute/virtualMachines
            location: centralus
   
          VM instance status
            code: ProvisioningState/succeeded
              level: Info
              displayStatus: Provisioning succeeded
            code: PowerState/running
              level: Info
              displayStatus: VM running

## <a name="stop-a-virtual-machine"></a>仮想マシンの停止
仮想マシンを停止するには、2 つの方法があります。 仮想マシンを停止してそのすべての設定を保持する (引き続き課金されます)、または仮想マシンを停止して割り当てを解除します。 仮想マシンの割り当てを解除すると、それに関連付けられているすべてのリソースの割り当ても解除され、仮想マシンに対する課金が終了します。

1. 前に Main メソッドに追加したコードを、資格情報を取得するコードを除いて、すべてコメントアウトします。
2. Program クラスに次のメソッドを追加します。
   
        public static async void StopVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Stopping the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          await computeManagementClient.VirtualMachines.PowerOffAsync(groupName, vmName);
        }
   
    仮想マシンの割り当てを解除する場合は、PowerOff 呼び出しを次のコードに変更します。
   
        computeManagementClient.VirtualMachines.Deallocate(groupName, vmName);
3. 追加したメソッドを呼び出すために、次のコードを Main メソッドに追加します。
   
        StopVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
4. Program.cs ファイルを保存します。
5. Visual Studio で **[開始]** をクリックし、サブスクリプションで使用するのと同じユーザー名とパスワードを使用して Azure AD にサインインします。
   
    仮想マシンの状態が [停止済み] に変化することを確認します。 Deallocate を呼び出すメソッドを実行した場合は、状態が [停止済み (割り当て解除)] になります。

## <a name="start-a-virtual-machine"></a>仮想マシンの起動
1. 前に Main メソッドに追加したコードを、資格情報を取得するコードを除いて、すべてコメントアウトします。
2. Program クラスに次のメソッドを追加します。
   
        public static async void StartVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Starting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          await computeManagementClient.VirtualMachines.StartAsync(groupName, vmName);
        }
3. 追加したメソッドを呼び出すために、次のコードを Main メソッドに追加します。
   
        StartVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
4. Program.cs ファイルを保存します。
5. Visual Studio で **[開始]** をクリックし、サブスクリプションで使用するのと同じユーザー名とパスワードを使用して Azure AD にサインインします。
   
    仮想マシンの状態が [実行中] に変化することを確認します。

## <a name="restart-a-running-virtual-machine"></a>仮想マシンの実行の再開
1. 前に Main メソッドに追加したコードを、資格情報を取得するコードを除いて、すべてコメントアウトします。
2. Program クラスに次のメソッドを追加します。
   
        public static async void RestartVirtualMachineAsync(
          TokenCredentials credential,
          string groupName,
          string vmName,
          string subscriptionId)
        {
          Console.WriteLine("Restarting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          await computeManagementClient.VirtualMachines.RestartAsync(groupName, vmName);
        }
3. 追加したメソッドを呼び出すために、次のコードを Main メソッドに追加します。
   
        RestartVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
4. Program.cs ファイルを保存します。
5. Visual Studio で **[開始]** をクリックし、サブスクリプションで使用するのと同じユーザー名とパスワードを使用して Azure AD にサインインします。

## <a name="resize-a-virtual-machine"></a>仮想マシンのサイズ変更
この例では、実行中の仮想マシンのサイズを変更する方法を示します。

1. 前に Main メソッドに追加したコードを、資格情報を取得するコードを除いて、すべてコメントアウトします。
2. Program クラスに次のメソッドを追加します。
   
        public static async void UpdateVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Updating the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var vmResult = await computeManagementClient.VirtualMachines.GetAsync(groupName, vmName);
          vmResult.HardwareProfile.VmSize = "Standard_A1";
          await computeManagementClient.VirtualMachines.CreateOrUpdateAsync(groupName, vmName, vmResult);
        }
3. 追加したメソッドを呼び出すために、次のコードを Main メソッドに追加します。
   
        UpdateVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
4. Program.cs ファイルを保存します。
5. Visual Studio で **[開始]** をクリックし、サブスクリプションで使用するのと同じユーザー名とパスワードを使用して Azure AD にサインインします。
   
    仮想マシンのサイズが [Standard_A1] に変化することを確認します。

## <a name="add-a-data-disk-to-a-virtual-machine"></a>データ ディスクを仮想マシンに追加する
この例では、実行中の仮想マシンにデータ ディスクを追加する方法を示します。

1. 前に Main メソッドに追加したコードを、資格情報を取得するコードを除いて、すべてコメントアウトします。
2. Program クラスに次のメソッドを追加します。
   
        public static async void AddDataDiskAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Adding the disk to the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          var vmResult = await computeManagementClient.VirtualMachines.GetAsync(groupName, vmName);
          vmResult.StorageProfile.DataDisks.Add(
            new DataDisk
              {
                Lun = 0,
                Name = "mydatadisk1",
                Vhd = new VirtualHardDisk
                  {
                    Uri = "https://mystorage1.blob.core.windows.net/vhds/mydatadisk1.vhd"
                  },
                CreateOption = DiskCreateOptionTypes.Empty,
                DiskSizeGB = 2,
                Caching = CachingTypes.ReadWrite
              });
          await computeManagementClient.VirtualMachines.CreateOrUpdateAsync(groupName, vmName, vmResult);
        }
3. 追加したメソッドを呼び出すために、次のコードを Main メソッドに追加します。
   
        AddDataDiskAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
4. Program.cs ファイルを保存します。
5. Visual Studio で **[開始]** をクリックし、サブスクリプションで使用するのと同じユーザー名とパスワードを使用して Azure AD にサインインします。

## <a name="delete-a-virtual-machine"></a>仮想マシンの削除
1. 前に Main メソッドに追加したコードを、資格情報を取得するコードを除いて、すべてコメントアウトします。
2. Program クラスに次のメソッドを追加します。
   
        public static async void DeleteVirtualMachineAsync(
          TokenCredentials credential, 
          string groupName, 
          string vmName, 
          string subscriptionId)
        {
          Console.WriteLine("Deleting the virtual machine...");
          var computeManagementClient = new ComputeManagementClient(credential)
            { SubscriptionId = subscriptionId };
          await computeManagementClient.VirtualMachines.DeleteAsync(groupName, vmName);
        }
3. 追加したメソッドを呼び出すために、次のコードを Main メソッドに追加します。
   
        DeleteVirtualMachineAsync(
          credential,
          groupName,
          vmName,
          subscriptionId);
        Console.WriteLine("\nPress enter to continue...");
        Console.ReadLine();
4. Program.cs ファイルを保存します。
5. Visual Studio で **[開始]** をクリックし、サブスクリプションで使用するのと同じユーザー名とパスワードを使用して Azure AD にサインインします。

## <a name="next-steps"></a>次のステップ
デプロイに問題がある場合は、「[Troubleshoot common Azure deployment errors with Azure Resource Manager](../azure-resource-manager/resource-manager-common-deployment-errors.md)」(Azure Resource Manager を使用した Azure のデプロイで発生する一般的なエラーのトラブルシューティング) を参照してください。




<!--HONumber=Jan17_HO2-->


