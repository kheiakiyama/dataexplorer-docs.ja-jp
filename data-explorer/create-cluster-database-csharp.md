---
title: C# を使用して Azure Data Explorer クラスターと DB を作成する
description: C# を使用して Azure Data Explorer クラスターとデータベースを作成する方法を学習します
author: orspod
ms.author: orspodek
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
ms.date: 06/03/2019
ms.openlocfilehash: 9182d4f5c12031cb4aaa62ab07bd00ab7864a627
ms.sourcegitcommit: a7458819e42815a0376182c610aba48519501d92
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/28/2020
ms.locfileid: "92902513"
---
# <a name="create-an-azure-data-explorer-cluster-and-database-by-using-c"></a>C# を使用して Azure Data Explorer クラスターとデータベースを作成する

> [!div class="op_single_selector"]
> * [ポータル](create-cluster-database-portal.md)
> * [CLI](create-cluster-database-cli.md)
> * [PowerShell](create-cluster-database-powershell.md)
> * [C#](create-cluster-database-csharp.md)
> * [Python](create-cluster-database-python.md)
> * [Go](create-cluster-database-go.md)
> * [Azure Resource Manager テンプレート](create-cluster-database-resource-manager.md)

Azure Data Explorer は、アプリケーション、Web サイト、IoT デバイスなどからの大量のデータ ストリーミングをリアルタイムに分析するためのフル マネージドのデータ分析サービスです。 Azure Data Explorer を使用するには、最初にクラスターを作成し、そのクラスター内に 1 つまたは複数のデータベースを作成します。 その後、クエリを実行できるように、データをデータベースに取り込み (読み込み) ます。 この記事では、C# を使用して、クラスターとデータベースを 1 つずつ作成します。

## <a name="prerequisites"></a>前提条件

* Visual Studio 2019 をインストールしていない場合は、 **無料** の [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/) をダウンロードして使用できます。 Visual Studio のセットアップ中に、必ず **[Azure の開発]** を有効にしてください。
* Azure サブスクリプションをお持ちでない場合は、開始する前に[無料の Azure アカウント](https://azure.microsoft.com/free/)を作成してください。

[!INCLUDE [data-explorer-data-connection-install-nuget-csharp](includes/data-explorer-data-connection-install-nuget-csharp.md)]

## <a name="authentication"></a>認証
この記事の例を実行するには、リソースにアクセスできる Azure AD アプリケーションとサービス プリンシパルが必要です。 「[Azure AD アプリケーションを作成する](/azure/active-directory/develop/howto-create-service-principal-portal)」を参照して、無料の Azure AD アプリケーションを作成し、サブスクリプション スコープでロールの割り当てを追加します。 `Directory (tenant) ID`、`Application ID`、および `Client Secret` を取得する方法も示されています。

## <a name="create-the-azure-data-explorer-cluster"></a>Azure Data Explorer クラスターを作成する

1. 次のコードを使用してクラスターを作成します。

    ```csharp
    var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
    var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
    var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
    var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";
    var authenticationContext = new AuthenticationContext($"https://login.windows.net/{tenantId}");
    var credential = new ClientCredential(clientId, clientSecret);
    var result = await authenticationContext.AcquireTokenAsync(resource: "https://management.core.windows.net/", clientCredential: credential);

    var credentials = new TokenCredentials(result.AccessToken, result.AccessTokenType);

    var kustoManagementClient = new KustoManagementClient(credentials)
    {
        SubscriptionId = subscriptionId
    };

    var resourceGroupName = "testrg";
    var clusterName = "mykustocluster";
    var location = "Central US";
    var skuName = "Standard_D13_v2";
    var tier = "Standard";
    var capacity = 5;
    var sku = new AzureSku(skuName, tier, capacity);
    var cluster = new Cluster(location, sku);
    await kustoManagementClient.Clusters.CreateOrUpdateAsync(resourceGroupName, clusterName, cluster);
    ```

   |**設定** | **推奨値** | **フィールドの説明**|
   |---|---|---|
   | clusterName | *mykustocluster* | クラスターの任意の名前。|
   | skuName | *Standard_D13_v2* | クラスターに使用される SKU。 |
   | レベル | *Standard* | SKU レベル。 |
   | capacity | *number* | クラスターのインスタンスの数。 |
   | resourceGroupName | *testrg* | クラスターが作成されるリソース グループの名前。 |

    > [!NOTE]
    > **クラスターの作成** は長時間実行される動作であるため、CreateOrUpdate の代わりに CreateOrUpdateAsync を使用することを強くお勧めします。 

1. クラスターが正常に作成されたかどうかを確認するには、次のコマンドを実行します。

    ```csharp
    kustoManagementClient.Clusters.Get(resourceGroupName, clusterName);
    ```

結果に値が `ProvisioningState` の `Succeeded` が含まれている場合、クラスターは正常に作成されています。

## <a name="create-the-database-in-the-azure-data-explorer-cluster"></a>Azure Data Explorer クラスターでデータベースを作成する

1. 次のコードを使用してデータベースを作成します。

    ```csharp
    var hotCachePeriod = new TimeSpan(3650, 0, 0, 0);
    var softDeletePeriod = new TimeSpan(3650, 0, 0, 0);
    var databaseName = "mykustodatabase";
    var database = new ReadWriteDatabase(location: location, softDeletePeriod: softDeletePeriod, hotCachePeriod: hotCachePeriod);

    await kustoManagementClient.Databases.CreateOrUpdateAsync(resourceGroupName, clusterName, databaseName, database);
    ```

    > [!NOTE]
    > C# バージョン 2.0.0 以前を使用している場合は、ReadWriteDatabase ではなく Database を使用します。

   |**設定** | **推奨値** | **フィールドの説明**|
   |---|---|---|
   | clusterName | *mykustocluster* | データベースの作成先となるクラスターの名前。|
   | databaseName | *mykustodatabase* | データベースの名前。|
   | resourceGroupName | *testrg* | クラスターが作成されるリソース グループの名前。 |
   | softDeletePeriod | *3650:00:00:00* | データをクエリに使用できるようにしておく時間。 |
   | hotCachePeriod | *3650:00:00:00* | データをキャッシュに保持する時間。 |

2. 次のコマンドを実行して、作成したデータベースを確認します。

    ```csharp
    kustoManagementClient.Databases.Get(resourceGroupName, clusterName, databaseName) as ReadWriteDatabase;
    ```

クラスターとデータベースが作成されました。

## <a name="clean-up-resources"></a>リソースをクリーンアップする

* 他の記事に進む場合は、作成したリソースをそのままにします。
* リソースをクリーンアップするには、クラスターを削除します。 クラスターを削除するときに、その中に含まれるデータベースもすべて削除されます。 クラスターを削除するには次のコマンドを使います。

    ```csharp
    kustoManagementClient.Clusters.Delete(resourceGroupName, clusterName);
    ```

## <a name="next-steps"></a>次のステップ

* [Azure Data Explorer .NET Standard SDK (プレビュー) を使用してデータを取り込む](./net-sdk-ingest-data.md)