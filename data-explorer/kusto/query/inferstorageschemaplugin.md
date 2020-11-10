---
title: infer_storage_schema プラグイン-Azure データエクスプローラー
description: この記事では、Azure データエクスプローラーの infer_storage_schema プラグインについて説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 895f12799f4c44346af2b6b9be43bf98b7cf870e
ms.sourcegitcommit: e820a59191d2ca4394e233d51df7a0584fa4494d
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/10/2020
ms.locfileid: "94446210"
---
# <a name="infer_storage_schema-plugin"></a>infer_storage_schema プラグイン

このプラグインは、外部データのスキーマを推測し、それを CSL スキーマ文字列として返します。 この文字列は、 [外部テーブルを作成](../management/external-tables-azurestorage-azuredatalake.md#create-or-alter-external-table)するときに使用できます。

```kusto
let options = dynamic({
  'StorageContainers': [
    h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
  ],
  'DataFormat': 'parquet',
  'FileExtension': '.parquet'
});
evaluate infer_storage_schema(options)
```

## <a name="syntax"></a>構文

`evaluate` `infer_storage_schema(` *Options* `)`

## <a name="arguments"></a>引数

単一 *オプション* の引数は、 `dynamic` 要求のプロパティを指定するプロパティバッグを保持する型の定数値です。

|名前                    |必須|説明|
|------------------------|--------|-----------|
|`StorageContainers`|はい|格納されているデータ成果物のプレフィックス URI を表す [ストレージ接続文字列](../api/connection-strings/storage.md) の一覧|
|`DataFormat`|はい|サポートされている [データ形式](../../ingestion-supported-formats.md)の1つ。|
|`FileExtension`|いいえ|このファイル拡張子で終わるファイルのみをスキャンします。 必須ではありませんが、これを指定するとプロセスが高速化されます (または、データの読み取りに関する問題を回避できます)。|
|`FileNamePrefix`|いいえ|このプレフィックスで始まるファイルのみをスキャンします。 必須ではありませんが、指定するとプロセスが高速化される可能性があります。|
|`Mode`|いいえ|スキーマの推論方法。次のいずれかに `any` `last` `all` なります。 (最初に検出された) ファイル、最後に書き込まれたファイル、またはすべてのファイルからデータスキーマを推論します。 既定値は `last` です。|

## <a name="returns"></a>戻り値

この `infer_storage_schema` プラグインは、CSL スキーマ文字列を保持する1つの行または列を含む単一の結果テーブルを返します。

> [!NOTE]
> * ストレージコンテナー URI の秘密キーには、 *読み取り* に加えて、 *リスト* に対するアクセス許可が必要です。
> * スキーマ推論戦略 ' all ' は、検出された *すべて* のアイテムからの読み取りを意味し、スキーマをマージするため、非常に "高額" な操作です。
> * 返される型の中には、誤った型推測 (または、スキーママージ処理の結果として) の結果として実際の型を使用できないものがあります。 このため、外部テーブルを作成する前に、結果を慎重に確認する必要があります。

## <a name="example"></a>例

```kusto
let options = dynamic({
  'StorageContainers': [
    h@'https://storageaccount.blob.core.windows.net/MovileEvents;secretKey'
  ],
  'FileExtension': '.parquet',
  'FileNamePrefix': 'part-',
  'DataFormat': 'parquet'
});
evaluate infer_storage_schema(options)
```

*結果*

|CslSchema|
|---|
|app_id: string、user_id: long、event_time: datetime、country: string、city: string、device_type: string、device_vendor: string、ad_network: string、: string、site_id: string、event_type: string、event_name: string、有機: string、days_from_install: int、収益: real|

外部テーブルの定義で返されたスキーマを使用します。

```kusto
.create external table MobileEvents(
    app_id:string, user_id:long, event_time:datetime, country:string, city:string, device_type:string, device_vendor:string, ad_network:string, campaign:string, site_id:string, event_type:string, event_name:string, organic:string, days_from_install:int, revenue:real
)
kind=blob
partition by (dt:datetime = bin(event_time, 1d), app:string = app_id)
pathformat = ('app=' app '/dt=' datetime_pattern('yyyyMMdd', dt))
dataformat = parquet
(
    h@'https://storageaccount.blob.core.windows.net/MovileEvents;secretKey'
)
```