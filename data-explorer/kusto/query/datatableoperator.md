---
title: datatable 演算子-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーでの datatable 演算子について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 6a5f12c947d62b7ec13ab6d9d6a4564881de08a2
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92247842"
---
# <a name="datatable-operator"></a>datatable 演算子

クエリ自体で定義されているスキーマと値を持つテーブルを返します。

> [!NOTE]
> この演算子にはパイプライン入力がありません。

## <a name="syntax"></a>構文

`datatable``(` *ColumnName* `:` *ColumnType* [ `,` ...] `)` `[` *ScalarValue* [ `,` *ScalarValue* ...]`]`

## <a name="arguments"></a>引数

::: zone pivot="azuredataexplorer"

* *ColumnName*、 *ColumnType*: これらの引数は、テーブルのスキーマを定義します。 引数には、テーブルを定義するときと同じ構文を使用します。
  詳細については、「 [create table](../management/create-table-command.md)」を参照してください。
* *ScalarValue*: テーブルに挿入する定数スカラー値。 値の数は、テーブル内の列の倍数の整数である必要があります。 *N*' 番目の値には、列*n*  %  *numcolumns*に対応する型を指定しなければなりません。

::: zone-end

::: zone pivot="azuremonitor"

* *ColumnName*、 *ColumnType*: これらの引数は、テーブルのスキーマを定義します。
* *ScalarValue*: テーブルに挿入する定数スカラー値。 値の数は、テーブル内の列の倍数の整数である必要があります。 *N*' 番目の値には、列*n*  %  *numcolumns*に対応する型を指定しなければなりません。

::: zone-end

## <a name="returns"></a>戻り値

この演算子は、指定されたスキーマとデータのデータテーブルを返します。

## <a name="example"></a>例

```kusto
datatable (Date:datetime, Event:string)
    [datetime(1910-06-11), "Born",
     datetime(1930-01-01), "Enters Ecole Navale",
     datetime(1953-01-01), "Published first book",
     datetime(1997-06-25), "Died"]
| where strlen(Event) > 4
```
