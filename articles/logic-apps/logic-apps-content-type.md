---
title: "Logic Apps のコンテンツ タイプの処理 | Microsoft Docs"
description: "設計時と実行時に Logic Apps によってコンテンツ タイプが処理される方法について説明します。"
services: logic-apps
documentationcenter: .net,nodejs,java
author: jeffhollan
manager: anneta
editor: 
ms.assetid: cd1f08fd-8cde-4afc-86ff-2e5738cc8288
ms.service: logic-apps
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: integration
ms.date: 10/18/2016
ms.author: jehollan
translationtype: Human Translation
ms.sourcegitcommit: 84f1968a4bda08232bb260f74915ef8889b53f3c
ms.openlocfilehash: 77daea3da5bdb8de374058ef7b1ab5926ed59aa7
ms.lasthandoff: 02/27/2017


---
# <a name="logic-apps-content-type-handling"></a>Logic Apps のコンテンツ タイプの処理
ロジック アプリには、JSON、XML、フラット ファイル、バイナリ データなど、さまざまな種類のコンテンツを渡すことができます。  すべてのコンテンツ タイプがサポートされていますが、Logic Apps エンジンでネイティブに理解されるものもあれば、キャストまたは変換が必要なものもあります。  次の記事では、エンジンがさまざまなコンテンツ タイプを処理する方法と、コンテンツ タイプが必要に応じて適切に処理されるしくみについて説明します。

## <a name="content-type-header"></a>Content-Type ヘッダー
まずは簡単なものから見ていきましょう。ロジック アプリでの使用時に変換やキャストを必要としない&2; つの `Content-Types`、つまり `application/json` と `text/plain` を確認します。

## <a name="applicationjson"></a>application/json
ワークフロー エンジンは、HTTP 呼び出しの `Content-Type` ヘッダーに基づいて適切な処理を決定します。  コンテンツ タイプが `application/json` の要求は、すべて JSON オブジェクトとして格納、処理されます。  また、JSON コンテンツはキャストしなくても既定で解析できます。  そのため、Content-Type ヘッダーが `application/json ` である要求は次のような形になります。

```
{
    "data": "a",
    "foo": [
        "bar"
    ]
}
```

この要求はワークフローで `@body('myAction')['foo'][0]` のような式で解析され、値 (この場合は `bar`) を取得できます。  追加のキャストは不要です。  JSON データを扱っているのに指定のヘッダーがない場合は、そのデータを `@json()` 関数 (例: `@json(triggerBody())['foo']`) を使用して手動で JSON にキャストできます。

### <a name="schema-and-schema-generator"></a>スキーマとスキーマ ジェネレーター
要求トリガーでは、受信を予定していたペイロードの JSON スキーマを入力することができます。 これにより、デザイナーで、要求のコンテンツを使用するためのトークンを生成できます。 スキーマの準備ができていない場合は、`Use sample payload to generate schema` を選択して、サンプル ペイロードから JSON スキーマを生成します。

![スキーマ](./media/logic-apps-http-endpoint/manualtrigger.png)

### <a name="use-parse-json-action"></a>Parse JSON アクションの使用
`Parse JSON` アクションでは、JSON コンテンツをわかりやすいトークンに解析し、ロジック アプリで使用するようにできます。 要求トリガーと同様に、解析するコンテンツの JSON スキーマを入力または生成することができます。 これは優れたツールで、Service Bus、DocumentDb などからのデータ利用がとても簡単になります。

![Parse JSON](./media/logic-apps-content-type/ParseJSON.png)

## <a name="textplain"></a>text/plain
`application/json` と同じように、`text/plain` の `Content-Type` ヘッダー付きで受信した HTTP メッセージは、未加工の形式で格納されます。  さらに、それが後続のアクションにキャストなしで含まれる場合、要求は `Content-Type`: `text/plain` ヘッダー付きで送信されます。  たとえば、フラット ファイルを処理している場合、次のような HTTP コンテンツを受信することがあります。

```
Date,Name,Address
Oct-1,Frank,123 Ave.
```

これは `text/plain`として受信します。  これを次のアクションで別の要求 (`@body('flatfile')`) の本文として送信した場合、この要求には `text/plain` の Content-Type ヘッダーが含まれることになります。  プレーン テキストのデータを扱っているのに指定のヘッダーがない場合は、そのデータを `@string()` 関数 (例: `@string(triggerBody())`) を使って手動でテキストにキャストできます。

## <a name="applicationxml-and-applicationoctet-stream-and-converter-functions"></a>Application/xml、Application/octet-stream、変換関数
ロジック アプリのエンジンは、HTTP 要求または HTTP 応答で受信した `Content-Type` を常に維持します。  つまり、`Content-Type` が `application/octet-stream` のコンテンツを受信した場合に、それを後続のアクションにキャストなしで含めると、`Content-Type`: `application/octet-stream` の要求として送信されます。  こうすることで、エンジンは、データがワークフローを移動する間に失われないようにすることができます。  ただし、アクションの状態 (入力と出力) は、ワークフローを移動する際に JSON オブジェクトに格納されます。  つまり、エンジンは一部のデータ型を維持するために、`$content` と `$content-type` の両方を保持する適切なメタデータ (自動的に変換される) を含む、Base64 でエンコードされたバイナリ文字列に変換します。  以下のように、組み込みの変換関数を使って、コンテンツ タイプを手動で変換することもできます。

* `@json()` - データを `application/json` にキャスト
* `@xml()` - データを `application/xml` にキャスト
* `@binary()` - データを `application/octet-stream` にキャスト
* `@string()` - データを `text/plain` にキャスト
* `@base64()` - コンテンツを Base64 文字列に変換
* `@base64toString()` - Base64 でエンコードされた文字列を `text/plain` に変換
* `@base64toBinary()` - Base64 でエンコードされた文字列を `application/octet-stream` に変換
* `@encodeDataUri()` - 文字列を Data URI のバイト配列としてエンコード
* `@decodeDataUri()` - Data URI をバイト配列にデコード

例として、次のような `Content-Type`: `application/xml` の HTTP 要求を受信したとします。

```
<?xml version="1.0" encoding="UTF-8" ?>
<CustomerName>Frank</CustomerName>
```

これは、`@xml(triggerBody())` などの関数や、`@xpath(xml(triggerBody()), '/CustomerName')` のような関数内で後でキャストして使用することができます。

## <a name="other-content-types"></a>他のコンテンツ タイプ
他のコンテンツ タイプもサポートされており、ロジック アプリで使用できますが、 `$content`をデコードして手動でメッセージ本文を取得しなければならない場合があります。  たとえば、次のような `application/x-www-url-formencoded` 要求をトリガーするとします。

```
CustomerName=Frank&Address=123+Avenue
```

これはプレーン テキストでも JSON でもないため、アクションに次のように格納されます。

```
...
"body": {
    "$content-type": "application/x-www-url-formencoded",
    "$content": "AAB1241BACDFA=="
}
```

このときの `$content` は、すべてのデータを保持するために Base64 文字列としてエンコードされたペイロードです。  現時点ではフォーム データ用のネイティブ関数は存在しないので、`@string(body('formdataAction'))` のような関数を使用して手動でデータにアクセスすれば、このデータをワークフロー内で使用できます。  送信要求に `application/x-www-url-formencoded` という Content-Type ヘッダーも加えたい場合は、単にアクション本体に追加すれば済みます。`@body('formdataAction')` のようなキャストは必要ありません。  ただし、これが有効なのは、本体が `body` 入力内で唯一のパラメーターであるときのみです。  `application/json` 要求内で `@body('formdataAction')` を実行しようとすると、エンコードされた本体が送信され、ランタイム エラーが発生します。
