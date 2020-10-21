---
title: assert ()-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの assert () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2019
ms.openlocfilehash: 442fbec2742a4d1edc7a9ad03a81db27e6d18574
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92252774"
---
# <a name="assert"></a>assert()

条件をチェックします。 条件が false の場合、ではエラーメッセージが出力され、クエリは失敗します。

## <a name="syntax"></a>構文

`assert(`*条件* `, `*メッセージ*`)`

## <a name="arguments"></a>引数

* *condition*: 評価する条件式。 条件がの場合 `false` 、指定されたメッセージはエラーを報告するために使用されます。 条件がの場合は `true` 、 `true` 評価結果としてを返します。 クエリ分析フェーズでは、条件を定数に評価する必要があります。
* *message*: アサーションがに評価される場合に使用される `false` メッセージ。 *メッセージ*は文字列リテラルである必要があります。

> [!NOTE]
> `condition` クエリ分析フェーズ中に定数に評価される必要があります。 言い換えると、定数を参照する他の式から構築することができ、行コンテキストにバインドすることはできません。

## <a name="returns"></a>戻り値

* `true` -条件がの場合 `true`
* 条件がに評価された場合にセマンティックエラーを発生させ `false` ます。

## <a name="examples"></a>例

次のクエリでは、 `checkLength()` 入力文字列の長さをチェックし、を使用して `assert` 入力長パラメーターを検証する関数を定義します (これが0より大きいことを確認します)。

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let checkLength = (len:long, s:string)
{
    assert(len > 0, "Length must be greater than zero") and 
    strlen(s) > len
};
datatable(input:string)
[
    '123',
    '4567'
]
| where checkLength(len=long(-1), input)
```

このクエリを実行すると、次のエラーが生成されます。  
`assert() has failed with message: 'Length must be greater than zero'`


有効な入力で実行する例 `len` :

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let checkLength = (len:long, s:string)
{
    assert(len > 0, "Length must be greater than zero") and strlen(s) > len
};
datatable(input:string)
[
    '123',
    '4567'
]
| where checkLength(len=3, input)
```

|input|
|---|
|4567|
