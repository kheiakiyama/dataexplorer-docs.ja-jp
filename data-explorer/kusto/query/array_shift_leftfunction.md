---
title: array_shift_left() - Azure データ エクスプローラー |マイクロソフトドキュメント
description: この記事では、Azure データ エクスプローラーで array_shift_left() について説明します。
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/11/2019
ms.openlocfilehash: 5b623bdcccc75261d0fb9324bb6015e16b0ff9b8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: ja-JP
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518787"
---
# <a name="array_shift_left"></a>array_shift_left()

`array_shift_left()`配列内の値を左にシフトします。

**構文**

`array_shift_left(`*arr,* *shift_count* [, *fill_value]*`)`

**引数**

* *arr*: 分割する入力配列は動的配列でなければなりません。
* *shift_count*: 配列要素を左にシフトする位置の数を指定する整数。 値が負の場合、要素は右にシフトされます。
* *fill_value*: シフトおよび削除された要素ではなく、要素を挿入するために使用されるスカラー値。 デフォルト: null 値または空の文字列 *(arr*の種類によって異なります)。

**戻り値**

元の配列と同じ量の要素を含む動的配列で、各要素は*shift_count*に従ってシフトされました。 削除される要素の代わりに追加される新しい要素は *、fill_value*値を持ちます。

**参照**

* 配列を右にシフトする場合は[、array_shift_right()](array_shift_rightfunction.md)を参照してください。
* 配列を右に回転させる場合は[、array_rotate_right()](array_rotate_rightfunction.md)を参照してください。
* 左の回転配列については[、array_rotate_left()](array_rotate_leftfunction.md)を参照してください。

**使用例**

* 2 つの位置で左にシフト:

    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_left(arr, 2)
    ```
    
    |Arr|arr_shift|
    |---|---|
    |[1,2,3,4,5]|[3,4,5,null,null]|

* 2 つの位置で左にシフトし、デフォルト値を追加します。

    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_left(arr, 2, -1)
    ```
    
    |Arr|arr_shift|
    |---|---|
    |[1,2,3,4,5]|[3,4,5,-1,-1]|


* 負の*shift_count*値を使用して、2 つの位置で右にシフトします。

    ```kusto
    print arr=dynamic([1,2,3,4,5]) 
    | extend arr_shift=array_shift_left(arr, -2, -1)
    ```
    
    |Arr|arr_shift|
    |---|---|
    |[1,2,3,4,5]|[-1,-1,1,2,3]|