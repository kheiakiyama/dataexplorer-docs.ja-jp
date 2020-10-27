---
title: Kibana を使用して Azure Data Explorer のデータを視覚化する
description: この記事では、Azure Data Explorer を Kibana のデータ ソースとして設定する方法について説明します
author: orspod
ms.author: orspodek
ms.reviewer: guregini
ms.service: data-explorer
ms.topic: how-to
ms.date: 03/12/2020
ms.openlocfilehash: d81ed37a7502e0795fc82f38a918719a5da8db8e
ms.sourcegitcommit: 898f67b83ae8cf55e93ce172a6fd3473b7c1c094
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/21/2020
ms.locfileid: "92342894"
---
# <a name="visualize-data-from-azure-data-explorer-in-kibana-with-the-k2bridge-open-source-connector"></a>K2Bridge オープンソース コネクタを使用して Kibana で Azure Data Explorer のデータを視覚化する

K2Bridge (Kibana-Kusto Bridge) では、データソースとして Azure Data Explorer を使用し、Kibana でそのデータを視覚化することができます。 K2Bridge は[オープンソース](https://github.com/microsoft/K2Bridge)のコンテナー化されたアプリケーションです。 Kibana インスタンスと Azure Data Explorer クラスター間のプロキシとして機能します。 この記事では、K2Bridge を使用してその接続を作成する方法について説明します。

K2Bridge では、Kibana クエリが Kusto Query Language (KQL) に変換されて、Azure Data Explorer の結果が Kibana に送り返されます。

   ![K2Bridge を介した Azure Data Explorer との Kibana 接続](media/k2bridge/k2bridge-chart.png)

K2Bridge は Kibana の **[Discover]\(検出\)** タブをサポートしており、このタブで次のことができます。

* データの検索と探索。
* 結果のフィルター処理。
* 結果グリッドでのフィールドの追加または削除。
* レコードの内容の表示。
* 検索の保存と共有。

次の図は、K2Bridge によって Azure Data Explorer にバインドされた Kibana インスタンスを示しています。 Kibana のユーザー エクスペリエンスは変更されません。

   [![Azure Data Explorer にバインドされた Kibana ページ](media/k2bridge/k2bridge-kibana-page.png)](media/k2bridge/k2bridge-kibana-page.png#lightbox)

## <a name="prerequisites"></a>前提条件

Kibana で Azure Data Explorer のデータを視覚化するには、事前に次の準備をしておく必要があります。

* [Helm V3](https://github.com/helm/helm#install)。Kubernetes パッケージ マネージャーです。

* Azure Kubernetes Service (AKS) クラスターまたはその他の Kubernetes クラスター。 バージョン 1.14 から 1.16 へのテストと検証が完了しました。 AKS クラスターが必要であれば、[Azure CLI を使用して](/azure/aks/kubernetes-walkthrough)、または [Azure portal を使用して](/azure/aks/kubernetes-walkthrough-portal)、AKS クラスターをデプロイする方法を参照してください。

* [Azure Data Explorer クラスター](create-cluster-database-portal.md)。クラスターの URL とデータベース名が含まれます。

* Azure Data Explorer でデータを表示することが承認された Azure Active Directory (Azure AD) サービス プリンシパル。クライアント ID とクライアント シークレットが含まれます。

    サービス プリンシパルに表示アクセス許可を設定することを推奨します。より高いレベルのアクセス許可は使用しないことをお勧めします。 [Azure AD サービス プリンシパルにクラスターの表示アクセス許可を設定します](manage-database-permissions.md#manage-permissions-in-the-azure-portal)。

    Azure AD サービス プリンシパルについて詳しくは、[Azure AD サービス プリンシパルの作成](/azure/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application)に関する記事をご覧ください。

## <a name="run-k2bridge-on-azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS) で K2Bridge を実行する

既定では、K2Bridges の Helm チャートでは、Microsoft の Container Registry (MCR) にある公開されているイメージが参照されます。 MCR は資格情報を必要としません。

1. 必要な Helm チャートをダウンロードします。

1. Elasticsearch 依存関係を Helm に追加します。 K2Bridge は内部 Elasticsearch のスモール インスタンスを使用するため、依存関係が必要です。 インスタンスは、インデックス パターン クエリや保存されたクエリなどの、メタデータ関連の要求を処理します。 この内部インスタンスは、ビジネス データを保存しません。 このインスタンスは実装の詳細情報であると考えることができます。

    1. Elasticsearch 依存関係を Helm に追加するには、次のコマンドを実行します。

        ```bash
        helm repo add elastic https://helm.elastic.co
        helm repo update
        ```

    1. K2Bridge チャートを GitHub リポジトリの charts ディレクトリから取得するには、以下を実行します。

        1. [GitHub](https://github.com/microsoft/K2Bridge) からリポジトリを複製します。
        1. K2Bridges ルート リポジトリ ディレクトリに移動します。
        1. 次のコマンドを実行します。

            ```bash
            helm dependency update charts/k2bridge
            ```

1. K2Bridge をデプロイします。

    1. 環境に合わせて変数に正しい値を設定します。

        ```bash
        ADX_URL=[YOUR_ADX_CLUSTER_URL] #For example, https://mycluster.westeurope.kusto.windows.net
        ADX_DATABASE=[YOUR_ADX_DATABASE_NAME]
        ADX_CLIENT_ID=[SERVICE_PRINCIPAL_CLIENT_ID]
        ADX_CLIENT_SECRET=[SERVICE_PRINCIPAL_CLIENT_SECRET]
        ADX_TENANT_ID=[SERVICE_PRINCIPAL_TENANT_ID]
        ```

    1. 必要に応じて、Application Insights テレメトリを有効にします。 Application Insights を初めて使用する場合は、[Application Insights リソースを作成](/azure/azure-monitor/app/create-new-resource)します。 変数に[インストルメンテーション キーをコピー](/azure/azure-monitor/app/create-new-resource#copy-the-instrumentation-key)します。

        ```bash
        APPLICATION_INSIGHTS_KEY=[INSTRUMENTATION_KEY]
        COLLECT_TELEMETRY=true
        ```

    1. <a name="install-k2bridge-chart"></a>K2Bridge チャートをインストールします。

        ```bash
        helm install k2bridge charts/k2bridge -n k2bridge --set image.repository=$REPOSITORY_NAME/$CONTAINER_NAME --set settings.adxClusterUrl="$ADX_URL" --set settings.adxDefaultDatabaseName="$ADX_DATABASE" --set settings.aadClientId="$ADX_CLIENT_ID" --set settings.aadClientSecret="$ADX_CLIENT_SECRET" --set settings.aadTenantId="$ADX_TENANT_ID" [--set image.tag=latest] [--set privateRegistry="$IMAGE_PULL_SECRET_NAME"] [--set settings.collectTelemetry=$COLLECT_TELEMETRY]
        ```

        「[構成](https://github.com/microsoft/K2Bridge/blob/master/docs/configuration.md)」では、構成オプションの完全なセットを見つけることができます。

    1. <a name="install-kibana-service"></a> 前のコマンドの出力では、Kibana をデプロイする次の Helm コマンドが提案されます。 必要に応じて、次のコマンドを実行します。

        ```bash
        helm install kibana elastic/kibana -n k2bridge --set image=docker.elastic.co/kibana/kibana-oss --set imageTag=6.8.5 --set elasticsearchHosts=http://k2bridge:8080
        ```

    1. ポート フォワーディングを使用して、localhost 上の Kibana にアクセスします。

        ```bash
        kubectl port-forward service/kibana-kibana 5601 --namespace k2bridge
        ```

    1. http://127.0.0.1:5601 に移動して、Kibana に接続します。

    1. Kibana をユーザーに公開します。 そのためには複数の方法があります。 使用する方法は、ユース ケースによって大きく異なります。

        たとえば、サービスをロード バランサー サービスとして公開できます。 これを行うには、 **--set service. type = LoadBalancer** パラメーターを [以前の Kibana Helm **インストール** コマンド](#install-kibana-service)に追加します。

        その後、次のコマンドを実行します。

        ```bash
        kubectl get service -w -n k2bridge
        ```

        出力は次のようになります。

        ```bash
        NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        kibana-kibana   LoadBalancer   xx.xx.xx.xx    <pending>     5601:30128/TCP   4m24s
        ```

        生成され表示された EXTERNAL-IP 値を使用できます。 これを使用して Kibana にアクセスするには、ブラウザーを開き、\<EXTERNAL-IP\>:5601 に移動します

1. データにアクセスするためのインデックス パターンを構成します。

    新しい Kibana インスタンスで次のようにします。

    1. Kibana を開きます。
    1. **[Management]\(管理\)** を参照します。
    1. **[Index Patterns]\(インデックス パターン\)** を選択します。
    1. インデックス パターンを作成します。 インデックスの名前は、テーブル名または関数名と完全に一致している必要があります。アスタリスク (\*) は使用できません。 リストから関連する行をコピーできます。

> [!Note]
> K2Bridge を他の Kubernetes プロバイダーで実行するには、Elasticsearch の values.yaml の **storageClassName** 値を、プロバイダーによって提案されたものに合うように変更します。

## <a name="visualize-data"></a>データの視覚化

Azure Data Explorer が Kibana のデータ ソースとして構成されている場合は、Kibana を使用してデータを探索できます。

1. Kibana の一番左側のメニューで、 **[Discover]\(検出\)** タブを選択します。

1. 一番左側のドロップダウン リスト ボックスから、インデックス パターンを選択します。 このパターンにより、探索するデータソースを定義します。 この場合、インデックス パターンは Azure Data Explorer テーブルです。

   ![インデックス パターンの選択](media/k2bridge/k2bridge-select-an-index-pattern.png)

1. データに時間フィルター フィールドがある場合は、時間の範囲を指定できます。 **[Discover]\(検出\)** ページの右上で、時間フィルターを選択します。 既定では、このページには過去 15 分間のデータが表示されます。

   ![時間フィルターの選択](media/k2bridge/k2bridge-time-filter.png)

1. 結果テーブルには、最初の 500 レコードが表示されます。 ドキュメントを展開して、JSON 形式またはテーブル形式のいずれかで、そのフィールド データを調べることができます。

   ![展開されたレコード](media/k2bridge/k2bridge-expand-record.png)

1. 既定では、結果テーブルには **_source** 列が含まれています。 また、時間フィールドが存在する場合は、 **Time** 列も含まれます。 一番左側のペインでフィールド名の横にある **[add]\(追加\)** を選択することで、結果テーブルに特定の列を追加できます。

   ![[add]\(追加\) ボタンが強調表示されている特定の列](media/k2bridge/k2bridge-specific-columns.png)

1. クエリ バーでは、次の方法でデータを検索できます。

    * 検索用語の入力。
    * Lucene クエリ構文の使用。 次に例を示します。
        * "error" を検索して、この値を含むすべてのレコードを検索します。
        * "status: 200" を検索して、状態値が 200 のすべてのレコードを取得します。
    * 論理演算子 **AND** 、 **OR** 、および **NOT** を使用します。
    * アスタリスク (\*) と疑問符 (?) のワイルドカード文字を使用します。 たとえば、クエリ "destination_city:L*" は、destination-city 値が "L" または "l" で始まるレコードと一致します。 (K2Bridge では大文字と小文字は区別されません。)

    ![Running a query](media/k2bridge/k2bridge-run-query.png)

    > [!Tip]
    > 「[検索](https://github.com/microsoft/K2Bridge/blob/master/docs/searching.md)」では、さらに多くの検索ルールとロジックを見つけることができます。

1. 検索結果をフィルター処理するには、ページの一番右側のペインにあるフィールド リストを使用します。 フィールド リストには、次の項目が表示されます。

    * フィールドの上位 5 つの値。
    * フィールドが含まれているレコードの数。
    * 各値を含むレコードの割合。

    >[!Tip]
    > 虫眼鏡を使用して、特定の値を持つすべてのレコードを検索します。

    ![虫眼鏡が強調表示されているフィールド リスト](media/k2bridge/k2bridge-field-list.png)

    また虫眼鏡を使用して結果をフィルター処理し、結果テーブル内の各レコードの結果テーブル形式ビューを表示することもできます。

     ![虫眼鏡が強調表示されているテーブル リスト](media/k2bridge/k2bridge-table-list.png)

1. 検索の **[Save]\(保存\)** または **[Share]\(共有\)** を選択します。

     ![検索を保存または共有するための強調表示されたボタン](media/k2bridge/k2bridge-save-search.png)