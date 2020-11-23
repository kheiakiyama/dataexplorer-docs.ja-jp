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
ms.openlocfilehash: 4b2a9d100cdb7125ebed21be69433fa3cee3a929
ms.sourcegitcommit: 4c7f20dfd59fb5b5b1adfbbcbc9b7da07df5e479
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/23/2020
ms.locfileid: "95324535"
---
# <a name="consume-operator"></a>consume 演算子

演算子に渡された表形式のデータストリームを使用します。 

演算子は、ほとんどの場合、 `consume` 結果を呼び出し元に返さずに、クエリの副作用をトリガーするために使用されます。

```kusto
T | consume
```

## <a name="syntax"></a>Syntax

`consume` [ `decodeblocks` `=` *DecodeBlocks*]

## <a name="arguments"></a>引数

* *DecodeBlocks*: 定数ブール値。 をに設定した場合、 `true` または要求プロパティがに設定されている場合、演算子はその入力のレコードを列挙するだけではなく、実際には、 `perftrace` `true` `consume` これらのレコード内の各値を強制的に圧縮解除およびデコードします。

演算子を使用すると、 `consume` 実際に結果をクライアントに返すことなく、クエリのコストを見積もることができます。
(推定はさまざまな理由で正確ではありません。たとえば、は distributively と計算されるため、 `consume` `T | consume` はクラスターのノード間でテーブルのデータを転送しません)。
