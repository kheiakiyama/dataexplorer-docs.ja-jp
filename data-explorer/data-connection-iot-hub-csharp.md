---
title: C# を使用して Azure Data Explorer 用に IoT Hub データ接続を作成する
description: この記事では、C# を使用して Azure Data Explorer 用に IoT Hub データ接続を作成する方法について学習します。
author: orspod
ms.author: orspodek
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: how-to
ms.date: 10/07/2019
ms.openlocfilehash: ff4a4f0ada7e12f5de239c1b78b39a8beace15ee
ms.sourcegitcommit: 898f67b83ae8cf55e93ce172a6fd3473b7c1c094
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342979"
---
# <a name="create-an-iot-hub-data-connection-for-azure-data-explorer-by-using-c-preview"></a>C# を使用して Azure Data Explorer 用に IoT Hub データ接続を作成する (プレビュー)

> [!div class="op_single_selector"]
> * [ポータル](ingest-data-iot-hub.md)
> * [C#](data-connection-iot-hub-csharp.md)
> * [Python](data-connection-iot-hub-python.md)
> * [Azure Resource Manager テンプレート](data-connection-iot-hub-resource-manager.md)

[!INCLUDE [data-connector-intro](includes/data-connector-intro.md)]
この記事では、C# を使用して Azure Data Explorer 用に IoT Hub データ接続を作成します。

## <a name="prerequisites"></a>前提条件

* Visual Studio 2019 をインストールしていない場合は、 **無料** の [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/) をダウンロードして使用できます。 Visual Studio のセットアップ中に、必ず **[Azure の開発]** を有効にしてください。
* Azure サブスクリプションをお持ちでない場合は、開始する前に[無料の Azure アカウント](https://azure.microsoft.com/free/)を作成してください。
* [クラスターとデータベース](create-cluster-database-csharp.md)を作成します
* [テーブルと列のマッピング](./net-sdk-ingest-data.md#create-a-table-on-your-test-cluster)を作成します
* [データベースとテーブルのポリシー](database-table-policies-csharp.md) (オプション) を設定します
* [共有アクセス ポリシーを構成して IoT Hub](ingest-data-iot-hub.md#create-an-iot-hub) を作成します。

[!INCLUDE [data-explorer-data-connection-install-nuget-csharp](includes/data-explorer-data-connection-install-nuget-csharp.md)]

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-an-iot-hub-data-connection"></a>IoT Hub データ接続を追加する 

次の例は、IoT Hub データ接続をプログラムを使用して追加する方法を示しています。 Azure portal を使用して Iot Hub データ接続を追加する方法については、「[Azure Data Explorer テーブルを IoT ハブに接続する](ingest-data-iot-hub.md#connect-azure-data-explorer-table-to-iot-hub)」を参照してください。

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
//The cluster and database that are created as part of the Prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
var dataConnectionName = "myeventhubconnect";
//The IoT hub that is created as part of the Prerequisites
var iotHubResourceId = "/subscriptions/xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxx/providers/Microsoft.Devices/IotHubs/xxxxxx";
var sharedAccessPolicyName = "iothubforread";
var consumerGroup = "$Default";
var location = "Central US";
//The table and column mapping are created as part of the Prerequisites
var tableName = "StormEvents";
var mappingRuleName = "StormEvents_CSV_Mapping";
var dataFormat = DataFormat.CSV;

await kustoManagementClient.DataConnections.CreateOrUpdate(resourceGroupName, clusterName, databaseName, dataConnectionName,
            new IotHubDataConnection(iotHubResourceId, consumerGroup, sharedAccessPolicyName, tableName: tableName, location: location, mappingRuleName: mappingRuleName, dataFormat: dataFormat));
```

|**設定** | **推奨値** | **フィールドの説明**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | テナント ID。 ディレクトリ ID とも呼ばれます。|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | リソースの作成に使用するサブスクリプション ID。|
| clientId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | ご利用のテナント内のリソースにアクセスできるアプリケーションのクライアント ID。|
| clientSecret | *xxxxxxxxxxxxxx* | ご利用のテナント内のリソースにアクセスできるアプリケーションのクライアント シークレット。 |
| resourceGroupName | *testrg* | ご利用のクラスターを含むリソース グループの名前。|
| clusterName | *mykustocluster* | ご利用のクラスターの名前。|
| databaseName | *mykustodatabase* | ご利用のクラスター内のターゲット データベースの名前。|
| dataConnectionName | *myeventhubconnect* | データ接続の任意の名前。|
| tableName | *StormEvents* | ターゲット データベース内のターゲット テーブルの名前。|
| mappingRuleName | *StormEvents_CSV_Mapping* | ターゲット テーブルに関連付けられている列マッピングの名前。|
| dataFormat | *csv* | メッセージのデータ形式。|
| iotHubResourceId | *リソース ID* | インジェスト用のデータを保持している IoT ハブのリソース ID。 |
| sharedAccessPolicyName | *iothubforread* | IoT Hub に接続するデバイスとサービスのアクセス許可を定義する共有アクセス ポリシーの名前。 |
| consumerGroup | *$Default* | ご利用のイベント ハブのコンシューマー グループ。|
| location | *米国中部* | データ接続リソースの場所。|

[!INCLUDE [data-explorer-data-connection-clean-resources-csharp](includes/data-explorer-data-connection-clean-resources-csharp.md)]