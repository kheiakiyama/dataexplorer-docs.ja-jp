---
title: current_principal ()-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの current_principal () について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: a2530b93bd5102b7c21aa535eb8e7ca7c4360ddf
ms.sourcegitcommit: 608539af6ab511aa11d82c17b782641340fc8974
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/20/2020
ms.locfileid: "92252496"
---
# <a name="current_principal"></a>current_principal()

::: zone pivot="azuredataexplorer"

クエリを実行する現在のプリンシパル名を返します。

## <a name="syntax"></a>構文

`current_principal()`

## <a name="returns"></a>戻り値

としての現在のプリンシパルの完全修飾名 (すべての形式) `string` 。  
文字列の形式は次のとおりです。  
*PrinciplaType* `=`*PrincipalId* `;`*TenantId*

## <a name="example"></a>例

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print fqn=current_principal()
```

|fqn|
|---|
|aaduser = 346e950e-4a62-42bf-96f5-4cf4eac3f11e; 72f988bf-86f1-41af-91ab-2d7cd011db47|

::: zone-end

::: zone pivot="azuremonitor"

この機能は、ではサポートされていません Azure Monitor

::: zone-end
