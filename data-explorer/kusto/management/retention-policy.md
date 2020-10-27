---
title: Kusto リテンション期間ポリシー管理-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの保持ポリシーについて説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 79cac49a553a2b906947b4c85948b67718641587
ms.sourcegitcommit: ef3d919dee27c030842abf7c45c9e82e6e8350ee
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/27/2020
ms.locfileid: "92630077"
---
# <a name="retention-policy-command"></a>アイテム保持ポリシー コマンド

この記事では、 [保持ポリシー](retentionpolicy.md)を作成および変更するために使用する制御コマンドについて説明します。

## <a name="show-retention-policy"></a>保持ポリシーの表示

```kusto
.show <entity_type> <database_or_table_or_materialized_view> policy retention

.show <entity_type> *  policy retention
```

* `entity_type` : テーブル、具体化したビュー、またはデータベース
* `database_or_table_or_materialized_view`: `database_name` また `database_name.table_name` は `table_name` (データベースコンテキストの場合) または `materialized_view_name`

**例**

という名前のデータベースの保持ポリシーを表示し `MyDatabase` ます。

```kusto
.show database MyDatabase policy retention
```

## <a name="delete-retention-policy"></a>保持ポリシーの削除

データ保持ポリシーの削除は、データの保持を無制限に設定する affectively です。

テーブルのデータ保持ポリシーを削除すると、テーブルはデータベースレベルから保持ポリシーを派生させます。

```kusto
.delete <entity_type> <database_or_table_or_materialized_view> policy retention
```

* `entity_type` : テーブル、具体化したビュー、またはデータベース
* `database_or_table_or_materialized_view`: `database_name` また `database_name.table_name` は `table_name` (データベースコンテキストの場合) または `materialized_view_name`

**例**

という名前のテーブルの保持ポリシーを削除し `MyTable1` ます。

```kusto
.delete table MyTable policy retention
```


## <a name="alter-retention-policy"></a>保持ポリシーの変更

```kusto
.alter <entity_type> <database_or_table_or_materialized_view> policy retention <retention_policy>

.alter tables (<table_name> [, ...]) policy retention <retention_policy>

.alter-merge <entity_type> <database_or_table_or_materialized_view> policy retention <retention_policy>

.alter-merge <entity_type> <database_or_table_or_materialized_view> policy retention [softdelete = <timespan>] [recoverability = disabled|enabled]
```

* `entity_type` : テーブルまたはデータベースまたは具体化ビュー
* `database_or_table_or_materialized_view`: `database_name` また `database_name.table_name` は `table_name` (データベースコンテキストの場合) または `materialized_view_name`
* `table_name` : データベースコンテキスト内のテーブルの名前。  ワイルドカード ( `*` ここでは許可されています)。
* `retention_policy` :

```kusto
    "{ 
        \"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Disabled\"
    }" 
```

**使用例**

という名前のデータベースの保持ポリシーを表示し `MyDatabase` ます。

```kusto
.show database MyDatabase policy retention
```

10日の論理的な削除期間と無効なデータの回復性を持つ保持ポリシーを設定します。

```kusto
.alter-merge table Table1 policy retention softdelete = 10d recoverability = disabled

.alter-merge materialized-view View1 policy retention softdelete = 10d recoverability = disabled
```

10日の論理的な削除期間で保持ポリシーを設定し、データの回復性を有効にします。

```kusto
.alter table Table1 policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"

.alter materialized-view View1 policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"
```

上記と同じ保持ポリシーを設定しますが、今度は複数のテーブル (Table1、Table2、および Table3) の場合は次のようにします。

```kusto
.alter tables (Table1, Table2, Table3) policy retention "{\"SoftDeletePeriod\": \"10.00:00:00\", \"Recoverability\": \"Enabled\"}"
```

ソフト削除期間と回復性が有効になっている場合、既定値の100年で保持ポリシーを設定します。

```kusto
.alter table Table1 policy retention "{}"
```