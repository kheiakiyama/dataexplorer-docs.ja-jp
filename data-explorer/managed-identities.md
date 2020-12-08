---
title: Azure Data Explorer クラスターのマネージド ID の構成方法
description: Azure Data Explorer クラスターのマネージド ID を構成する方法について説明します。
author: orspod
ms.author: orspodek
ms.reviewer: itsagui
ms.service: data-explorer
ms.topic: how-to
ms.date: 11/25/2020
ms.openlocfilehash: 2bfc2cd69d88395e04af16e732564fb90170ee7f
ms.sourcegitcommit: 7edce9d9d20f9c0505abda67bb8cc3d2ecd60d15
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 12/02/2020
ms.locfileid: "96524285"
---
# <a name="configure-managed-identities-for-your-azure-data-explorer-cluster"></a>Azure Data Explorer クラスターのマネージド ID の構成

[Azure Active Directory のマネージド ID](/azure/active-directory/managed-identities-azure-resources/overview) を使用すると、クラスターは Azure AD で保護された他のリソース (Azure Key Vault など) に簡単にアクセスできます。 ID は Azure プラットフォームによって管理され、シークレットをプロビジョニングまたはローテーションする必要はありません。 マネージド ID の構成は、現在、[お使いのクラスターに対して、お客様が管理するキーを有効にする](security.md#customer-managed-keys-with-azure-key-vault)場合にのみサポートされています。

Azure Data Explorer クラスターには、次の 2 種類の ID を付与できます。

* **システム割り当て ID**:クラスターに関連付けられ、リソースが削除された場合は削除されます。 クラスターは 1 つのシステム割り当て ID しか持つことはできません。
* **ユーザー割り当て ID**:クラスターに割り当てることができるスタンドアロン Azure リソース。 クラスターは複数のユーザー割り当て ID を持つことができます。

この記事では、Azure Data Explorer のシステム割り当ておよびユーザー割り当てのマネージド ID を追加および削除する方法について説明します。

> [!Note]
> Azure Data Explorer クラスターがサブスクリプションやテナント間で移行された場合、Azure Data Explorer のマネージド ID は想定されたとおりに動作しません。 アプリは、機能を[無効にして](#remove-a-system-assigned-identity)から[再度有効にする](#add-a-system-assigned-identity)ことで、新しい ID を取得する必要があります。 新しい ID を使用するには、ダウンストリーム リソースのアクセス ポリシーも更新する必要があります。

## <a name="add-a-system-assigned-identity"></a>システム割り当て ID を追加する

クラスターに関連付けられており、クラスターが削除されると削除されるシステム割り当て ID を割り当てます。 クラスターは 1 つのシステム割り当て ID しか持つことはできません。 システム割り当て ID を持つクラスターを作成するには、クラスター上で追加のプロパティを設定する必要があります。 以下に詳細を示すとおり、Azure portal、C#、または Resource Manager テンプレートを使用してシステム割り当て ID を追加します。

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

### <a name="add-a-system-assigned-identity-using-the-azure-portal"></a>Azure portal を使用してシステム割り当て ID を追加する

1. [Azure portal](https://portal.azure.com/) にサインインします。

#### <a name="new-azure-data-explorer-cluster"></a>新しい Azure Data Explorer クラスター

1. [Azure Data Explorer クラスターを作成します](create-cluster-database-portal.md#create-a-cluster) 
1. **[セキュリティ]** タブ > **[システム割り当て ID]** で、 **[オン]** を選択します。 システム割り当て ID を削除するには、 **[オフ]** を選択します。
1. **[次へ :タグ] >** または **[確認と作成]** を選択して、クラスターを作成します。

    ![システム割り当て ID を新しいクラスターに追加する](media/managed-identities/system-assigned-identity-new-cluster.png)

#### <a name="existing-azure-data-explorer-cluster"></a>既存の Azure Data Explorer クラスター

1. 既存の Azure Data Explorer クラスターを開きます。
1. ポータルの左ペインで、 **[設定]**  >  **[ID]** を選択します。
1. **[ID]** ペイン > **[システム割り当て]** タブで、次のようにします。
   1. **[状態]** スライダーを **[オン]** に移動します。
   1. **[保存]** を選びます。
   1. ポップアップ ウィンドウで、 **[はい]** を選択します。

    ![システム割り当て ID を追加する](media/managed-identities/turn-system-assigned-identity-on.png)

1. 数分後、画面に以下が表示されます。
    * **オブジェクト ID** - カスタマー マネージド キーに使用されます
    * **アクセス許可** - 関連するロールの割り当てを選択します

    ![システム割り当て ID がオン](media/managed-identities/system-assigned-identity-on.png)

# <a name="c"></a>[C#](#tab/c-sharp)

### <a name="add-a-system-assigned-identity-using-c"></a>C# を使用してシステム割り当て ID を追加する

#### <a name="prerequisites"></a>前提条件

Azure Data Explorer C# クライアントを使用してマネージド ID を設定するには、次のようにします。

* [Azure Data Explorer (Kusto) NuGet パッケージ](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/)をインストールします。
* 認証用に、[Microsoft.IdentityModel.Clients.ActiveDirectory NuGet パッケージ](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)をインストールします。
* リソースにアクセスできる [Azure AD アプリケーション](/azure/active-directory/develop/howto-create-service-principal-portal)とサービス プリンシパルを作成します。 サブスクリプションのスコープでロールの割り当てを追加し、必要な `Directory (tenant) ID`、`Application ID`、および `Client Secret` を取得します。

#### <a name="create-or-update-your-cluster"></a>クラスターを作成または更新します

1. `Identity` プロパティを使用してクラスターを作成または更新します。

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
    var identity = new Identity(IdentityType.SystemAssigned);
    var cluster = new Cluster(location, sku, identity: identity);
    await kustoManagementClient.Clusters.CreateOrUpdateAsync(resourceGroupName, clusterName, cluster);
    ```

2. 次のコマンドを実行して、クラスターが ID とともに正常に作成または更新されたかどうかを確認します。

    ```csharp
    kustoManagementClient.Clusters.Get(resourceGroupName, clusterName);
    ```

    結果の `ProvisioningState` の値が `Succeeded` であれば、クラスターが作成または更新されたので、次のプロパティを持つはずです。

    ```csharp
    var principalId = cluster.Identity.PrincipalId;
    var tenantId = cluster.Identity.TenantId;
    ```

`PrincipalId` と `TenantId` は GUID で置き換えられます。 `TenantId` プロパティは、ID が属している Azure AD テナントを特定します。 `PrincipalId` は、クラスターの新しい ID の一意識別子です。 Azure AD のサービス プリンシパル名は、お使いの App Service または Azure Functions のインスタンスに指定したものと同じです。

# <a name="resource-manager-template"></a>[Resource Manager テンプレート](#tab/arm)

### <a name="add-a-system-assigned-identity-using-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートを使用して、システム割り当て ID を追加する

Azure Resource Manager テンプレートを使って、Azure リソースのデプロイを自動化できます。 Azure Data Explorer へのデプロイの詳細については、「[Azure Resource Manager テンプレートを使用して Azure Data Explorer クラスターとデータベースを作成する](create-cluster-database-resource-manager.md)」を参照してください。

システム割り当てタイプを追加することで、Azure に対してクラスターの ID を作成して管理するように指示します。 種類が `Microsoft.Kusto/clusters` であるすべてのリソースは、リソース定義に次のプロパティを含めることにより、ID を使って作成できます。

```json
"identity": {
    "type": "SystemAssigned"
}
```

次に例を示します。

```json
{
    "apiVersion": "2019-09-07",
    "type": "Microsoft.Kusto/clusters",
    "name": "[variables('clusterName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "SystemAssigned"
    }
}
```

> [!NOTE]
> クラスターは、システム割り当て ID とユーザー割り当て ID の両方を同時に持つことができます。 `type` プロパティは `SystemAssigned,UserAssigned` になります。

クラスターが作成されると、次の追加プロパティを持ちます。

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

`<TENANTID>` と `<PRINCIPALID>` は GUID で置き換えられます。 `TenantId` プロパティは、ID が属している Azure AD テナントを特定します。 `PrincipalId` は、クラスターの新しい ID の一意識別子です。 Azure AD のサービス プリンシパル名は、お使いの App Service または Azure Functions のインスタンスに指定したものと同じです。

---

## <a name="remove-a-system-assigned-identity"></a>システム割り当て ID を削除する

システム割り当て ID を削除すると、Azure AD からも削除されます。 クラスター リソースが削除されると、システム割り当て ID も Azure AD から自動的に削除されます。 システム割り当て ID を削除するには、機能を無効にします。 以下に詳細を示すとおり、Azure portal、C#、または Resource Manager テンプレートを使用してシステム割り当て ID を削除します。

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

### <a name="remove-a-system-assigned-identity-using-the-azure-portal"></a>Azure portal を使用してシステム割り当て ID を削除する

1. [Azure portal](https://portal.azure.com/) にサインインします。
1. ポータルの左ペインで、 **[設定]**  >  **[ID]** を選択します。
1. **[ID]** ペイン > **[システム割り当て]** タブで、次のようにします。
    1. **[状態]** スライダーを **[オフ]** に移動します。
    1. **[保存]** を選びます。
    1. ポップアップ ウィンドウで **[はい]** を選択して、システム割り当て ID を無効にします。 **[ID]** ペインは、システム割り当て ID が追加される前と同じ状態に戻ります。

    ![システム割り当て ID がオフ](media/managed-identities/system-assigned-identity.png)

# <a name="c"></a>[C#](#tab/c-sharp)

### <a name="remove-a-system-assigned-identity-using-c"></a>C# を使用してシステム割り当て ID を削除する

システム割り当て ID を削除するには、次を実行します。

```csharp
var identity = new Identity(IdentityType.None);
var cluster = new Cluster(location, sku, identity: identity);
await kustoManagementClient.Clusters.CreateOrUpdateAsync(resourceGroupName, clusterName, cluster);
```

# <a name="resource-manager-template"></a>[Resource Manager テンプレート](#tab/arm)

### <a name="remove-a-system-assigned-identity-using-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートを使用して、システム割り当て ID を削除する

システム割り当て ID を削除するには、次を実行します。

```json
"identity": {
    "type": "None"
}
```

> [!NOTE]
> クラスターにシステム割り当てとユーザー割り当て ID の両方が同時にある場合、システム割り当て ID の削除後、`type` プロパティは `UserAssigned` になります。

---

## <a name="add-a-user-assigned-identity"></a>ユーザー割り当て ID を追加する

クラスターにユーザー割り当てマネージド ID を割り当てます。 クラスターは複数のユーザー割り当て ID を持つことができます。 ユーザー割り当て ID を持つクラスターを作成するには、クラスター上で追加のプロパティを設定する必要があります。 以下に詳細を示すとおり、Azure portal、C#、または Resource Manager テンプレートを使用してユーザー割り当て ID を追加します。

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

### <a name="add-a-user-assigned-identity-using-the-azure-portal"></a>Azure portal を使用してユーザー割り当て ID を追加する

1. [Azure portal](https://portal.azure.com/) にサインインします。
1. [ユーザー割り当てマネージド ID リソースを作成します](/azure/active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal#create-a-user-assigned-managed-identity)。
1. 既存の Azure Data Explorer クラスターを開きます。
1. ポータルの左ペインで、 **[設定]**  >  **[ID]** を選択します。
1. **[ユーザー割り当て済み]** タブで、 **[追加]** を選択します。
1. 先ほど作成した ID を検索して選択します。 **[追加]** を選択します。

    ![ユーザー割り当て ID を追加する](media/managed-identities/user-assigned-identity-select.png)

# <a name="c"></a>[C#](#tab/c-sharp)

### <a name="add-a-user-assigned-identity-using-c"></a>C# を使用してユーザー割り当て ID を追加する

#### <a name="prerequisites"></a>前提条件

Azure Data Explorer C# クライアントを使用してマネージド ID を設定するには、次のようにします。

* [Azure Data Explorer (Kusto) NuGet パッケージ](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/)をインストールします。
* 認証用に、[Microsoft.IdentityModel.Clients.ActiveDirectory NuGet パッケージ](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/)をインストールします。
* リソースにアクセスできる [Azure AD アプリケーション](/azure/active-directory/develop/howto-create-service-principal-portal)とサービス プリンシパルを作成します。 サブスクリプションのスコープでロールの割り当てを追加し、必要な `Directory (tenant) ID`、`Application ID`、および `Client Secret` を取得します。

#### <a name="create-or-update-your-cluster"></a>クラスターを作成または更新します

1. `Identity` プロパティを使用してクラスターを作成または更新します。

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
    var identityName = "myIdentity";
    var userIdentityResourceId = $"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}";
    var userAssignedIdentities = new Dictionary<string, IdentityUserAssignedIdentitiesValue>(1) { { userIdentityResourceId, new IdentityUserAssignedIdentitiesValue() } };
    var identity = new Identity(type: IdentityType.UserAssigned, userAssignedIdentities: userAssignedIdentities);
    var cluster = new Cluster(location, sku, identity: identity);
    await kustoManagementClient.Clusters.CreateOrUpdateAsync(resourceGroupName, clusterName, cluster);
    ```

2. 次のコマンドを実行して、クラスターが ID とともに正常に作成または更新されたかどうかを確認します。

    ```csharp
    kustoManagementClient.Clusters.Get(resourceGroupName, clusterName);
    ```

    結果の `ProvisioningState` の値が `Succeeded` であれば、クラスターが作成または更新されたので、次のプロパティを持つはずです。

    ```csharp
    var userIdentity = cluster.Identity.UserAssignedIdentities[userIdentityResourceId];
    var principalId = userIdentity.PrincipalId;
    var clientId = userIdentity.ClientId;
    ```

`PrincipalId` は、Azure AD の管理に使用される ID の一意識別子です。 `ClientId` は、ランタイム呼び出し中に使用される ID を指定するために使用される、アプリケーションの新しい ID の一意識別子です。

# <a name="resource-manager-template"></a>[Resource Manager テンプレート](#tab/arm)

### <a name="add-a-user-assigned-identity-using-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートを使用して、ユーザー割り当て ID を追加する

Azure Resource Manager テンプレートを使って、Azure リソースのデプロイを自動化できます。 Azure Data Explorer へのデプロイの詳細については、「[Azure Resource Manager テンプレートを使用して Azure Data Explorer クラスターとデータベースを作成する](create-cluster-database-resource-manager.md)」を参照してください。

種類が `Microsoft.Kusto/clusters` であるリソースは、リソース定義に次のプロパティを含めて、`<RESOURCEID>` を目的の ID のリソース ID と置き換えることで、ユーザー割り当て ID を使って作成できます。

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {}
    }
}
```

次に例を示します。

```json
{
    "apiVersion": "2019-09-07",
    "type": "Microsoft.Kusto/clusters",
    "name": "[variables('clusterName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]": {}
        }
    },
    "dependsOn": [
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
    ]
}
```

クラスターが作成されると、次の追加プロパティを持ちます。

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {
            "principalId": "<PRINCIPALID>",
            "clientId": "<CLIENTID>"
        }
    }
}
```

`PrincipalId` は、Azure AD の管理に使用される ID の一意識別子です。 `ClientId` は、ランタイム呼び出し中に使用される ID を指定するために使用される、アプリケーションの新しい ID の一意識別子です。

> [!NOTE]
> クラスターは、システム割り当て ID とユーザー割り当て ID の両方を同時に持つことができます。 この場合、`type` プロパティは `SystemAssigned,UserAssigned` になります。

---

## <a name="remove-a-user-assigned-managed-identity-from-a-cluster"></a>クラスターからユーザー割り当てマネージド ID を削除する

以下に詳細を示すとおり、Azure portal、C#、または Resource Manager テンプレートを使用してユーザー割り当て ID を削除します。

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

### <a name="remove-a-user-assigned-managed-identity-using-the-azure-portal"></a>Azure portal を使用して、ユーザー割り当てマネージド ID を削除する

1. [Azure portal](https://portal.azure.com/) にサインインします。
1. ポータルの左ペインで、 **[設定]**  >  **[ID]** を選択します。
1. **[ユーザー割り当て済み]** タブを選択します。
1. 先ほど作成した ID を検索して選択します。 **[削除]** を選択します。

    ![ユーザー割り当て ID を削除する](media/managed-identities/user-assigned-identity-remove.png)

1. ポップアップ ウィンドウで **[はい]** を選択して、ユーザー割り当て ID を削除します。 **[ID]** ペインは、ユーザー割り当て ID が追加される前と同じ状態に戻ります。

# <a name="c"></a>[C#](#tab/c-sharp)

### <a name="remove-a-user-assigned-identity-using-c"></a>C# を使用してユーザー割り当て ID を削除する

ユーザー割り当て ID を削除するには、次を実行します。

```csharp
var identityName = "myIdentity";
var userIdentityResourceId = $"/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identityName}";
var userAssignedIdentities = new Dictionary<string, IdentityUserAssignedIdentitiesValue>(1) { { userIdentityResourceId, null } };
var identity = new Identity(type: IdentityType.UserAssigned, userAssignedIdentities: userAssignedIdentities);
var cluster = new Cluster(location, sku, identity: identity);
await kustoManagementClient.Clusters.CreateOrUpdateAsync(resourceGroupName, clusterName, cluster);
```

# <a name="resource-manager-template"></a>[Resource Manager テンプレート](#tab/arm)

### <a name="remove-a-user-assigned-identity-using-an-azure-resource-manager-template"></a>Azure Resource Manager テンプレートを使用して、ユーザー割り当て ID を削除する

ユーザー割り当て ID を削除するには、次を実行します。

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": null
    }
}
```

> [!NOTE]
>
> * ID を削除するには、値を null 値に設定します。 その他のすべての既存の ID は影響を受けません。
> * すべてのユーザー割り当て ID を削除するには、`type` プロパティを `None` にします。
> * クラスターにシステム割り当てとユーザー割り当ての両方の ID が同時にある場合、`type` プロパティは削除する ID により `SystemAssigned,UserAssigned` となるか、またはすべてのユーザー割り当て ID を削除する場合は `SystemAssigned` となります。

---

## <a name="next-steps"></a>次のステップ

* [Azure で Azure Data Explorer クラスターをセキュリティで保護する](security.md)
* 保存時の暗号化を有効にすることで、[Azure Data Explorer のディスク暗号化を使用してクラスターをセキュリティで保護する - Azure portal](cluster-disk-encryption.md)。
* [C# を使用してカスタマー マネージド キーを構成する](customer-managed-keys-csharp.md)
* [Azure Resource Manager テンプレートを使用してカスタマー マネージド キーを構成する](customer-managed-keys-resource-manager.md)
