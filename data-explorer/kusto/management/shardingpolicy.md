---
title: Data シャーディング policy-Azure データエクスプローラー |Microsoft Docs
description: この記事では、Azure データエクスプローラーの Data シャーディング policy について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 70d4013c8524fa88249de14a0a67cc8a85e73b3f
ms.sourcegitcommit: 898f67b83ae8cf55e93ce172a6fd3473b7c1c094
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/21/2020
ms.locfileid: "92337569"
---
# <a name="data-sharding-policy"></a>データシャーディングポリシー

シャーディングポリシーは、Azure データエクスプローラークラスターの [エクステント (データシャード)](../management/extents-overview.md) を封印する必要があるかどうかを定義します。

> [!NOTE]
> このポリシーは、新しいエクステントを作成するすべての操作に適用されます。たとえば、 [データの取り込み](../../ingest-data-overview.md#kusto-query-language-ingest-control-commands)用のコマンドや、 [. merge コマンドや rebuild コマンドなどです。](./merge-extents.md)

Data シャーディング policy には、次のプロパティが含まれています。

- **Maxrowcount**:
    - インジェストまたは再構築操作によって作成されたエクステントの最大行数。
    - 既定値は75万です。
    - [マージ操作](mergepolicy.md)では**無効**です。
        - マージ操作によって作成されるエクステントの行数を制限する必要がある場合は、 `RowCountUpperBoundForMerge` エンティティの [エクステントのマージポリシー](mergepolicy.md)でプロパティを調整します。
- **MaxExtentSizeInMb**:
    - マージ操作によって作成されたエクステントに許可される最大圧縮データサイズ (メガバイト単位)。
    - **[マージ](mergepolicy.md)操作に対してのみ**有効です。
    - 既定値は 1024 (1 GB) です。

- **Maxoriginalsizeinmb**:
    - 再構築操作によって作成されたエクステントに許可される元のデータの最大サイズ (mb 単位)。
    - **[再構築](mergepolicy.md)操作に対してのみ**有効です。
    - 既定値は 2048 (2 GB) です。

> [!WARNING]
> データシャーディングポリシーを変更する前に、Azure データエクスプローラーチームに相談してください。

作成されたデータベースには、既定の data シャーディングポリシーが含まれています。 このポリシーは、データベースで作成されたすべてのテーブルによって継承されます (ただし、ポリシーがテーブルレベルで明示的にオーバーライドされている場合を除く)。

[シャーディング policy control コマンド](../management/sharding-policy.md)) を使用すると、データベースとテーブルのデータシャーディングポリシーを管理できます。
