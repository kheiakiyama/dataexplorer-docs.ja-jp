---
title: Kusto IngestionBatching ポリシーによるバッチ処理の最適化-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの IngestionBatching ポリシーについて説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 27c92b9081844df0e8f8d4dc207ba5d2a8a2e2f3
ms.sourcegitcommit: a10e7c6ba96bdb94d95ef23f5d1506eb8fda0041
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/14/2020
ms.locfileid: "92058667"
---
# <a name="ingestionbatching-policy"></a>IngestionBatching ポリシー

## <a name="overview"></a>概要

インジェスト処理中に、インジェストを待機している間に小さな受信データチャンクを一括してバッチ処理することで、スループットの最適化を試みます。
この種のバッチ処理により、インジェストプロセスによって消費されるリソースが減少するだけでなく、バッチを使用しないインジェストによって生成される小さなデータシャードを最適化するための取り込み後のリソースは必要ありません。

ただし、インジェストの前にバッチ処理を実行することには欠点があります。これは、強制的な遅延が発生するので、クエリの準備が整うまでデータのインジェストを要求することからエンドツーエンドの時間を要求することになります。

このトレードオフを制御できるようにするには、ポリシーを使用し `IngestionBatching` ます。
このポリシーは、キューに置かれたインジェストにのみ適用され、小さい blob をまとめてバッチ処理するときに許容される最大の強制遅延を提供します。

## <a name="details"></a>詳細

前に説明したように、データの最適なサイズは一括で取り込まれたます。
現時点では、このサイズは約 1 GB の非圧縮データです。 最適なサイズよりもはるかに少ないデータを保持する blob でのインジェストは最適ではないため、キューに置かれたインジェストでは、このような小さな blob をまとめてバッチ処理します。 バッチ処理は、最初の条件が true になるまで実行されます。

1. バッチ処理されたデータの合計サイズが最適なサイズに達した場合、または
2. ポリシーで許容される最大遅延時間、合計サイズ、または blob の数に `IngestionBatching` 達しました

ポリシーは、 `IngestionBatching` データベースまたはテーブルに対して設定できます。 既定では、ポリシーが定義されていない場合、Kusto は既定値の **5 分** を使用します。これは、バッチ処理の最大遅延時間 **1000** 項目、合計サイズ **1g** です。

> [!WARNING]
> このポリシーを非常に小さい値に設定することによる影響は、クラスターの COGS が増加し、パフォーマンスが低下することにあります。 さらに、この制限により、この値を小さくすると、複数のインジェストプロセスを並行して管理するオーバーヘッドにより、エンドツーエンドのインジェスト遅延が実質的に **向上** する可能性があります。

## <a name="additional-resources"></a>その他の資料

* [IngestionBatching ポリシーコマンドのリファレンス](../management/batching-policy.md)
* [インジェストのベストプラクティス-スループットの最適化](../api/netfx/kusto-ingest-best-practices.md#optimizing-for-throughput)
