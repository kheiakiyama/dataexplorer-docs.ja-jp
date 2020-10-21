---
title: 使用演算子-Azure データエクスプローラー |Microsoft Docs
description: この記事では、Azure データエクスプローラーの使用演算子について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 1e4b2ff4ee45fd92e7e16f9468c18e1d0a70ca82
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92245159"
---
# <a name="consume-operator"></a>consume 演算子

演算子に渡された表形式のデータストリームを使用します。 

演算子は、ほとんどの場合、 `consume` 結果を呼び出し元に返さずに、クエリの副作用をトリガーするために使用されます。

```kusto
T | consume
```

## <a name="syntax"></a>構文

`consume` [ `decodeblocks` `=` *DecodeBlocks*]

## <a name="arguments"></a>引数

* *DecodeBlocks*: 定数ブール値。 をに設定した場合、 `true` または要求プロパティがに設定されている場合、演算子はその入力のレコードを列挙するだけではなく、実際には、 `perftrace` `true` `consume` これらのレコード内の各値を強制的に圧縮解除およびデコードします。

演算子を使用すると、 `consume` 実際に結果をクライアントに返すことなく、クエリのコストを見積もることができます。
(推定はさまざまな理由で正確ではありません。たとえば、は distributively と計算されるため、 `consume` `T | consume` はクラスターのノード間でテーブルのデータを転送しません)。

<!--
* *WithStats*: A constant Boolean value. If set to `true` (or if the global
  property `perftrace` is set), the operator will return a single
  row with a single column called `Stats` of type `dynamic` holding the statistics
  of the data source fed to the `consume` operator.
-->