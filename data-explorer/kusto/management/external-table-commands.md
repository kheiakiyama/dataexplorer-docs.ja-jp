---
title: Kusto External table 全般制御コマンド-Azure データエクスプローラー
description: この記事では、一般的な外部テーブルコントロールコマンドについて説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/26/2020
ms.openlocfilehash: 7646f86c9a521ab45cf83d7704084f3a5df6b256
ms.sourcegitcommit: 80f0c8b410fa4ba5ccecd96ae3803ce25db4a442
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/30/2020
ms.locfileid: "96321388"
---
# <a name="external-table-general-control-commands"></a>外部テーブル全般制御コマンド

外部テーブルの概要については、「 [外部テーブル](../query/schema-entities/externaltables.md) 」を参照してください。 次のコマンドは _、任意の外部テーブル_ (任意の型) に関連しています。

## <a name="show-external-tables"></a>。外部テーブルを表示します。

* データベース内のすべての外部テーブル (または特定の外部テーブル) を返します。
* [データベースモニターのアクセス許可](../management/access-control/role-based-authorization.md)が必要です。

**構文 :** 

`.show` `external` `tables`

`.show` `external` `table` *TableName*

**出力**

| 出力パラメーター | 種類   | 説明                                                         |
|------------------|--------|---------------------------------------------------------------------|
| TableName        | string | 外部テーブルの名前                                             |
| TableType        | string | 外部テーブルの種類                                              |
| Folder           | string | テーブルのフォルダー                                                     |
| DocString        | string | テーブルをドキュメント化する文字列                                       |
| プロパティ       | string | テーブルの JSON でシリアル化されたプロパティ (テーブルの型に固有) |


**例:**

```kusto
.show external tables
.show external table T
```

| TableName | TableType | Folder         | DocString | プロパティ |
|-----------|-----------|----------------|-----------|------------|
| T         | BLOB      | ExternalTables | Docs      | {}         |


## <a name="show-external-table-schema"></a>。外部テーブルスキーマを表示します

* JSON または CSL として、外部テーブルのスキーマを返します。 
* [データベースモニターのアクセス許可](../management/access-control/role-based-authorization.md)が必要です。

**構文 :** 

`.show``external` `table` *TableName* `schema` `as` ( `json`  |  `csl` )

`.show` `external` `table` *TableName* `cslschema`

**出力**

| 出力パラメーター | 種類   | 説明                        |
|------------------|--------|------------------------------------|
| TableName        | string | 外部テーブルの名前            |
| スキーマ           | string | JSON 形式のテーブルスキーマ |
| DatabaseName     | string | テーブルのデータベース名             |
| Folder           | string | テーブルのフォルダー                    |
| DocString        | string | テーブルをドキュメント化する文字列      |

**例:**

```kusto
.show external table T schema as JSON
```

```kusto
.show external table T schema as CSL
.show external table T cslschema
```

**出力:**

*json*

| TableName | スキーマ    | DatabaseName | Folder         | DocString |
|-----------|----------------------------------|--------------|----------------|-----------|
| T         | {"Name": "ExternalBlob",<br>"Folder": "ExternalTables",<br>"DocString": "Docs",<br>"OrderedColumns": [{"Name": "x", "Type": "system.string", "DocString", "Type": "system.string", "CslType": "string", "DocString": ""}]} を指定すると、"{" Name ":" x "," Type ":" system.object "," " | DB           | ExternalTables | Docs      |


*csl*

| TableName | スキーマ          | DatabaseName | Folder         | DocString |
|-----------|-----------------|--------------|----------------|-----------|
| T         | x:long、-string | DB           | ExternalTables | Docs      |

## <a name="drop-external-table"></a>。外部テーブルを削除します。

* 外部テーブルを削除します。 
* この操作後、外部テーブルの定義を復元できません
* [Database admin 権限](../management/access-control/role-based-authorization.md)が必要です。

**構文 :**  

`.drop``external` `table` *TableName* [ `ifexists` ]

**出力**

削除されたテーブルのプロパティを返します。 詳細については、[`.show external tables`](#show-external-tables) をご覧ください。

**例:**

```kusto
.drop external table ExternalBlob
```

| TableName | TableType | Folder         | DocString | スキーマ       | プロパティ |
|-----------|-----------|----------------|-----------|-----------------------------------------------------|------------|
| T         | BLOB      | ExternalTables | Docs      | [{"Name": "x", "CslType": "long"},<br> {"Name": "s", "CslType": "string"}] | {}         |

## <a name="next-steps"></a>次の手順

* [Azure Storage または Azure Data Lake の外部テーブルを作成および変更する](external-tables-azurestorage-azuredatalake.md)
* [外部 SQL テーブルを作成および変更する](external-sql-tables.md)
