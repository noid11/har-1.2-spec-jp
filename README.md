# HAR 1.2 Spec 日本語訳

[HAR 1.2 Spec](http://www.softwareishard.com/blog/har-12-spec/) の日本語訳です。
問題やフィードバックがあれば [Issue](https://github.com/noid11/har-1.2-spec-jp/issues) でご連絡ください。


# HAR 1.2 Spec

この文書は、HTTP モニタリングツールが収集したデータをエクスポートするために使用できる HTTP Archive 1.2 フォーマットを説明することを目的としています。


## HTTP Archive v1.2

HTTP Archive フォーマットの目標の 1つ は、十分に柔軟性を持たせることで、プロジェクトや様々なツールに採用できるようにすることです。
これにより、様々なソースからのデータを効率的に処理および分析できるようになるはずです。
なお、生成される HAR ファイルには、プライバシーやセキュリティに関わる機密データが含まれる可能性があるため、ユーザーエージェントは、このファイルを他者に転送する前に、その事実をユーザーに通知する方法を見つける必要があります。

- 以下で説明されるフォーマットは HTTP Archive 1.1 に基づいています
- このフォーマットは [JSON](http://www.ietf.org/rfc/rfc4627.txt) に基づいています
- [ニュースグループ](http://groups.google.com/group/http-archive-specification?hl=en)でフォローアップしてください
- [オンライン HAR ビューアー](http://www.softwareishard.com/blog/har-viewer/)が利用可能です
- 問題があれば [issue list](http://code.google.com/p/http-archive-specification/issues/list) で報告してください
- [HAR をサポートしているツール一覧](http://www.softwareishard.com/blog/har-adopters/)をご確認ください


## HAR Data Structure

HAR ファイルは UTF-8 エンコーディングで保存する必要があり、他のエンコーディングは禁じられています。
この仕様では、ツールが BOM をサポートし、それを無視することを要求しています。
また、ツールが必要に応じて BOM を出力することも許可されています。

HAR オブジェクトタイプのサマリ

- [log](#log)
- [creator](#creator)
- [browser](#browser)
- [pages](#pages)
- [pageTimings](#pagetimings)
- [entries](#entries)
- [request](#request)
- [response](#response)
- [cookies](#cookies)
- [headers](#headers)
- [queryString](#querystring)
- [postData](#postdata)
- [params](#params)
- [content](#content)
- [cache](#cache)
- [timings](#timings)


### log

このオブジェクトは、エクスポートされたデータのルートを表します。

```json
{
    "log": {
        "version" : "1.2",
        "creator" : {},
        "browser" : {},
        "pages": [],
        "entries": [],
        "comment": ""
    }
}
```

- version [string] - フォーマットのバージョン番号。空の場合、デフォルトで "1.1" という文字列が想定されます
- creator [object] - ログ作成アプリケーションの名前とバージョン情報
- browser [object, optional] - 使用されたブラウザの名前とバージョン
- pages [array, optional] - エクスポートされた全てのページのリスト。アプリケーションがページごとのグループ化をサポートしていない場合、このフィールドを省略します
- entries [array] - エクスポートされた全てのリクエストのリスト
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント

エクスポートされたウェブページごとに 1つ の `<page>` オブジェクトがあり、 HTTP リクエストごとに 1つ の `<entry>` オブジェクトがあります。
HTTP トレースツールがリクエストのページごとにグループ化出来ない場合、 `<pages>` オブジェクトは空であり、個々のリクエストには親ページがありません。


### creator

- creator と browser オブジェクトは同じ構造を共有しています

```json
"creator": {
    "name": "Firebug",
    "version": "1.6",
    "comment": ""
}
```


### browser

```json
"browser": {
    "name": "Firefox",
    "version": "3.6",
    "comment": ""
}
```

- name [string] - ログのエクスポートに使用されたアプリケーション/ブラウザの名前
- version [string] - ログのエクスポートに使用されたアプリケーション/ブラウザのバージョン
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント


### pages

このオブジェクトは、エクスポートされたページのリストを表します。

```json
"pages": [
    {
        "startedDateTime": "2009-04-16T12:07:25.123+01:00",
        "id": "page_0",
        "title": "Test Page",
        "pageTimings": {...},
        "comment": ""
    }
```

- startedDateTime [string] - ページの読み込みが始まった日時のタイムスタンプ (ISO 8601形式 - YYYY-MM-DDThh:mm.ss.sTZD, 例: 2009-07-24T19:20:30.45+01:00)
- id [string] - `<log>` 内でページを一意に識別する識別子。エントリはこれを使用して親ページを参照します
- title [string] - ページのタイトル
- pageTimings [object] - ページの読み込みに関する詳細なタイミング情報
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント


### pageTimings

このオブジェクトは、ページ読込中に発生する様々なイベントのタイミングを説明します。
すべての時間はミリ秒で指定されています。
タイミング情報が利用できない場合、該当するフィールドには `-1` が設定されます。

```json
"pageTimings": {
    "onContentLoad": 1720,
    "onLoad": 2500,
    "comment": ""
}
```

- onContentLoad [number, optional] - ページのコンテンツが読み込まれた時間。ページ読み込みが開始 (page.startedDateTime) されてからのミリ秒数。このタイミングが現在のリクエストに適用されない場合は `-1` を使用します
- onLoad [number, optional] - ページが読み込まれた (onLoad イベントが発生した) 時間。ページ読み込みが開始 (page.startedDateTime) されてからのミリ秒数。このタイミングが現在のリクエストに適用されない場合は `-1` を使用します
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント

ブラウザによっては `onContentLoad` プロパティは `DOMContentLoad` イベントまたは `document.readyState == interactive` を表します。


### entries

このオブジェクトは、エクスポートされた全ての HTTP リクエストを含む配列を表します。
`startedDateTime` でエントリを古い順からソートしてデータをエクスポートすることが推奨されます。
これにより、インポートがより速く行える可能性があるためです。
ただし、リーダーアプリケーションは、必要に応じて常に配列がソートされていることを確認する必要があります。

```json
"entries": [
    {
        "pageref": "page_0",
        "startedDateTime": "2009-04-16T12:07:23.596Z",
        "time": 50,
        "request": {...},
        "response": {...},
        "cache": {...},
        "timings": {},
        "serverIPAddress": "10.0.0.1",
        "connection": "52492",
        "comment": ""
    }
]
```

- pageref [string, unique, optional] - 親ページへの参照。アプリケーションがページごとのグループ化をサポートしていない場合は、このフィールドを省略します
- startedDateTime [string] - リクエスト開始時の日時のタイムスタンプ(ISO 8601形式 - YYYY-MM-DDThh:mm:ss.sTZD)
- time [number] - リクエストの合計経過時間 (ミリ秒)。これは、 timings オブジェクトに利用可能なすべてのタイミングの合計です()`-1` の値は含まれません)
- request [object] - リクエストに関する詳細情報
- response [object] - レスポンスに関する詳細情報
- cache [object] - キャッシュ使用に関する情報
- timings [object] - リクエスト/レスポンスの往復に関する詳細なタイミング情報
- serverIPAddress [string, optional] (new in 1.2) - 接続されたサーバーの IP アドレス (DNS 解決の結果)。
- connection [string, optional] (new in 1.2) - 親 TCP/IP 接続の一意の ID、クライアントまたはサーバーのポート番号である可能性があります。ただし、ポート番号が一意の識別子ではない場合 (複数の接続でポートが共有されている場合)、アプリケーションにポートが利用できない場合は、別の一意の接続 ID (例: 接続インデックス) が使用される可能性があります。この情報をアプリケーションがサポートしていない場合は、このフィールドを省略します
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント。


### request

このオブジェクトには、実行されたリクエストに関する詳細な情報が含まれています。

```json
"request": {
    "method": "GET",
    "url": "http://www.example.com/path/?param=value",
    "httpVersion": "HTTP/1.1",
    "cookies": [],
    "headers": [],
    "queryString" : [],
    "postData" : {},
    "headersSize" : 150,
    "bodySize" : 0,
    "comment" : ""
}
```

- method [string] - リクエストメソッド (GET, POST など)
- url [string] - リクエストの絶対 URL (フラグメントは含まれません)
- httpVersion [string] - リクエストの HTTP バージョン
- cookies [array] - クッキーオブジェクトのリスト
- headers [array] - ヘッダーオブジェクトのリスト
- queryString [array] - クエリパラメータオブジェクトのリスト
- postData [object, optional] - POST データ情報
- headersSize [number] - HTTP リクエストメッセージの開始から、ボディ前となる 2重 の CRLF を含むまでのバイト数の合計。情報が利用できない場合は `-1` に設定します
- bodySize [number] - リクエストボディ (POST データペイロード) のサイズ (バイト数)。情報が利用できない場合は `-1` に設定します。
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント

送信されたリクエストの総サイズは、次のように計算できます (両方の値が利用可能な場合):

```js
var totalSize = entry.request.headersSize + entry.request.bodySize;
```


### response

このオブジェクトには、レスポンスに関する詳細な情報が含まれています。

```json
"response": {
    "status": 200,
    "statusText": "OK",
    "httpVersion": "HTTP/1.1",
    "cookies": [],
    "headers": [],
    "content": {},
    "redirectURL": "",
    "headersSize" : 160,
    "bodySize" : 850,
    "comment" : ""
}
```

- status [number] - レスポンスのステータス
- statusText [string] - レスポンスステータスの説明
- httpVersion [string] - レスポンスの HTTP バージョン
- cookies [array] - クッキーオブジェクトのリスト
- headers [array] - ヘッダーオブジェクトのリスト
- content [object] - レスポンスボディに関する詳細
- redirectURL [string] - Location レスポンスヘッダーからのリダイレクト先 URL
- *headersSize [number] ** - HTTP レスポンスメッセージの開始から、ボディ前となる 2重 の CRLF を含むまでのバイト数の合計。情報が利用できない場合は `-1` に設定します
- bodySize [number] - 受信したレスポンスボディのサイズ (バイト数)。キャッシュからのレスポンス (304) の場合はゼロに設定します。情報が利用できない場合は `-1` に設定します
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント

** 受信したレスポンスヘッダーのサイズは、実際にサーバーから受信したヘッダーのみを基に計算されます。ブラウザによって追加されたヘッダーはこの数値には含まれませんが、ヘッダーオブジェクトのリストには表示されます。

受信したレスポンスの総サイズは、次のように計算できます (両方の値が利用可能な場合)：

```js
var totalSize = entry.response.headersSize + entry.response.bodySize;
```


### cookies

This object contains list of all cookies (used in <request> and <response> objects).
このオブジェクトには、すべてのクッキーのリストが含まれています (`request` および `response` オブジェクトで使用されています)。

```json
"cookies": [
    {
        "name": "TestCookie",
        "value": "Cookie Value",
        "path": "/",
        "domain": "www.janodvarko.cz",
        "expires": "2009-07-24T19:20:30.123+02:00",
        "httpOnly": false,
        "secure": false,
        "comment": ""
    }
]
```

- name [string] - クッキーの名前
- value [string] - クッキーの値
- path [string, optional] - クッキーに関連するパス
- domain [string, optional] - クッキーのホスト
- expires [string, optional] - クッキーの有効期限 (ISO 8601形式 - YYYY-MM-DDThh:mm:ss.sTZD、例: 2009-07-24T19:20:30.123+02:00)
- httpOnly [boolean, optional] - クッキーが HTTP Only であれば true そうでなければ false
- secure [boolean, optional] (new in 1.2) - クッキーが SSL 経由で送信された場合はtrue そうでなければ false
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント


### headers

このオブジェクトには、すべてのヘッダーのリストが含まれています (`request` および `response` オブジェクトで使用されています)。

```json
"headers": [
    {
        "name": "Accept-Encoding",
        "value": "gzip,deflate",
        "comment": ""
    },
    {
        "name": "Accept-Language",
        "value": "en-us,en;q=0.5",
        "comment": ""
    }
]
```


### queryString

このオブジェクトには、クエリストリングからパースされた全てのパラメーターと値のリストが含まれています (`request` オブジェクトに埋め込まれています)。

```json
"queryString": [
    {
        "name": "param1",
        "value": "value1",
        "comment": ""
    },
    {
        "name": "param1",
        "value": "value1",
        "comment": ""
    }
]
```

HAR フォーマットでは、クエリストリングのフォーマットとして NVP (name-value ペア) 形式が期待されています。


### postData

このオブジェクトは、 POST されたデータについて説明します (`request` オブジェクトに埋め込まれています)。

```json
"postData": {
    "mimeType": "multipart/form-data",
    "params": [],
    "text" : "plain posted data",
    "comment": ""
}
```

- mimeType [string] - POST されたデータの MIME タイプ
- params [array] - POST されたパラメータのリスト (URL エンコードされたパラメータの場合)
- text [string] - POST されたプレーンテキストデータ
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント

`text` フィールドと `params` フィールドは相互排他的です。


### params

POST されたパラメーターのリスト (`postData` オブジェクトに埋め込まれています)。

```json
"params": [
    {
        "name": "paramName",
        "value": "paramValue",
        "fileName": "example.pdf",
        "contentType": "application/pdf",
        "comment": ""
    }
]
```

- name [string] - POST されたパラメータの名前
- value [string, optional] - POST されたパラメータの値、または POST されたファイルの内容
- fileName [string, optional] - POST されたファイルの名前
- contentType [string, optional] - POST されたファイルのコンテンツタイプ
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント


### content

このオブジェクトは、レスポンス内容に関する詳細を説明します (`response` オブジェクトに埋め込まれています)。

```json
"content": {
    "size": 33,
    "compression": 0,
    "mimeType": "text/html; charset=utf-8",
    "text": "\n",
    "comment": ""
}
```

- size [number] - 返されたコンテンツの長さ (バイト単位)。圧縮がない場合は `response.bodySize` と等しく、コンテンツが圧縮されている場合はそれより大きくなります
- compression [number, optional] - 節約されたバイト数。情報が利用できない場合は、このフィールドを省略します
- mimeType [string] - レスポンステキストの MIME タイプ (Content-Type レスポンスヘッダーの値)。MIME タイプの charset 属性も含まれます (利用可能な場合)
- text [string, optional] - サーバーから送信された、またはブラウザキャッシュから読み込まれたレスポンスボディ。このフィールドには、テキストコンテンツのみが入力されます。`text` フィールドは HTTP デコードされたテキストか、またはレスポンスボディのエンコードされた表現 (例: "base64") です。情報が利用できない場合は、このフィールドを省略します
- encoding [string, optional] (new in 1.2) - レスポンステキストフィールドに使用されるエンコーディング。例: "base64"。 `text` フィールドが HTTP デコードされている場合（デコンプレッションおよびチャンク解除され、元の文字セットから UTF-8 にトランスコードされている場合）、このフィールドは省略します
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント

`text` フィールドが設定される前に、 HTTP レスポンスはデコードされ (解凍およびチャンク解除され)、その後、元の文字セットから UTF-8 にトランスコードされます。
さらに、 base64 などを使用してエンコードすることも可能です。
理想的には、アプリケーションは base64 エンコードされたデータをデコードし、ブラウザが操作したものとバイト単位で同一のリソースを取得できるようにするべきです。

`encoding` フィールドは、バイナリレスポンス (例: 画像) を HAR ファイルに含めるために役立ちます。

以下は、エンコードされたレスポンスの別の例です。元のレスポンス:


```html
<html><head></head><body/></html>\n
```

```json
"content": {
    "size": 33,
    "compression": 0,
    "mimeType": "text/html; charset=utf-8",
    "text": "PGh0bWw+PGhlYWQ+PC9oZWFkPjxib2R5Lz48L2h0bWw+XG4=",
    "encoding": "base64",
    "comment": ""
}
```


### cache

このオブジェクトには、ブラウザのキャッシュからのリクエストに関する情報が含まれています。

```json
"cache": {
    "beforeRequest": {},
    "afterRequest": {},
    "comment": ""
}
```

- beforeRequest [object, optional] - リクエスト前のキャッシュエントリの状態。情報が利用できない場合は、このフィールドを省略します
- afterRequest [object, optional] - リクエスト後のキャッシュエントリの状態。情報が利用できない場合は、このフィールドを省略します
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント

キャッシュ情報が利用できない場合、オブジェクトは以下のようになります (またはフィールド全体を省略することもできます)。

```json
"cache": {}
```

リクエスト前のキャッシュエントリに関する情報が利用できず、リクエスト後にもキャッシュエントリが存在しない場合、オブジェクトは以下のようになります。

```json
"cache": {
    "afterRequest": null
}
```

リクエスト前にもリクエスト後にもキャッシュエントリが存在しない場合、オブジェクトは以下のようになります。

```json
"cache": {
    "beforeRequest": null,
    "afterRequest": null
}
```

エントリがキャッシュに存在せず、リクエストによってコンテンツがダウンロードされた後にキャッシュされたことを示すために、オブジェクトは以下のようになります。

```json
"cache": {
    "beforeRequest": null,
    "afterRequest": {
        "expires": "2009-04-16T15:50:36",
        "lastAccess": "2009-16-02T15:50:34",
        "eTag": "",
        "hitCount": 0,
        "comment": ""
    }
}
```

`beforeRequest` と `afterRequest` のオブジェクトは、以下の構造を共有します。

```json
"beforeRequest": {
    "expires": "2009-04-16T15:50:36",
    "lastAccess": "2009-16-02T15:50:34",
    "eTag": "",
    "hitCount": 0,
    "comment": ""
}
```

- expires [string, optional] - キャッシュエントリの有効期限
- lastAccess [string] - キャッシュエントリが最後にアクセスされた時刻
- eTag [string] - Etag
- hitCount [number] - キャッシュエントリが開かれた回数
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント


### timings

このオブジェクトは、 request - response の往復における様々なフェーズを説明しています。
すべての時間はミリ秒で指定されています。

```json
"timings": {
    "blocked": 0,
    "dns": -1,
    "connect": 15,
    "send": 20,
    "wait": 38,
    "receive": 12,
    "ssl": -1,
    "comment": ""
}
```

- blocked [number, optional] - ネットワーク接続を待機するキューで費やされた時間。現在のリクエストに該当しない場合は `-1` を使用します
- dns [number, optional] - DNS 解決時間。ホスト名の解決に必要な時間。現在のリクエストに該当しない場合は `-1` を使用します
- connect [number, optional] - TCP 接続を作成するために必要な時間。現在のリクエストに該当しない場合は `-1` を使用します
- send [number] - HTTP リクエストをサーバーに送信するために必要な時間
- wait [number] - サーバーからのレスポンスを待機する時間
- receive [number] - サーバー(またはキャッシュ) からレスポンス全体を読み取るために必要な時間
- ssl [number, optional] (new in 1.2) - SSL/TLS ネゴシエーションに必要な時間。このフィールドが定義されている場合、 `connect` フィールドにもこの時間が含まれます (HAR 1.1 との互換性を確保するため)。現在のリクエストに該当しない場合は `-1` を使用します
- comment [string, optional] (new in 1.2) - ユーザーまたはアプリケーションによって提供されたコメント

`send`, `wait`, `receive` のタイミングはオプションではなく、負でない値を持たなければなりません。

エクスポートツールは、タイミングを提供できない場合、 `blocked`, `dns`, `connect`, `ssl` のタイミングを各リクエストで省略できます。
これらのタイミングを提供できるツールは、該当しない場合、値を `-1` に設定することができます。
例えば、既存の接続を再利用するリクエストでは、 `connect` は `-1` になります。

リクエストの時間値は、このセクションで提供されるタイミングの合計に等しくなければなりません (`-1`の値を除く)。

`-1` の値がない場合、次のことが ture でなければなりません (`entry` は `log.entries` 内のオブジェクトです)：

```js
entry.time == entry.timings.blocked + entry.timings.dns +
    entry.timings.connect + entry.timings.send + entry.timings.wait +
    entry.timings.receive;
```


## Custom Fields

仕様では、出力形式に新しいカスタムフィールドを追加することが許可されています。
以下のルールが適用されます: 

- カスタムフィールドおよび要素は、必ずアンダースコアで始めなければなりません (仕様のフィールドはアンダースコアで始めるべきではありません)。
- パーサーは、ファイルが読み込んでいるツールと同じツールによって書き込まれていない場合、すべてのカスタムフィールドおよび要素を無視しなければなりません
- パーサーは、マイナーバージョン番号がパーサーが対応している最大マイナーバージョンより大きい場合、解析方法がわからない非カスタムフィールドを無視しなければなりません
- パーサーは、特定の仕様バージョンには存在しなかったとわかっている非カスタムフィールドを含むファイルを拒否することができます


## Versioning Scheme

仕様番号には次の構文があります:

```
<major-version-number>.<minor-version-number>
```

メジャーバージョンは全体的な後方互換性を示し、マイナーバージョンは漸進的な変更を示します。
そのため、仕様に後方互換性がある変更が加えられた場合、マイナーバージョンが増加します。既存のフィールドに互換性のない変更が必要な場合、メジャーバージョンが増加します (例: 2.0)。

例:
```
1.2 -> 1.3
1.111 -> 1.112 (111 に更に変更がある場合)
1.5 -> 2.0 (2.0 は 1.5 との互換性を持ちません)
```

そのため、ツールが HAR 1.1 以降をサポートしている場合、次の構造を使用して互換性のないバージョンを検出できます。

```js
if (majorVersion != 1 || minorVersion < 1)
{
    throw "Incompatible version";
}
```

この例では、ツールはバージョンが 0.8, 0.9, 1.0 の場合に例外を投げますが、 1.1, 1.2, 1.112 などのバージョンでは動作します。
バージョン 2.x は拒否されます。


## HAR With Padding

JSONP (パディング付き JSON) に対するサポートは、 HAR のコア仕様の一部ではありません。
しかし、 HAR ファイルをオンラインで利用するための非常に優れた機能を提供します。

HAR ファイルをオンラインで提供するためには、 URL にコールバック URL パラメーターを含めることで、 HAR を関数へのコールバックとしてラップする必要があります (パディング付きの HAR ファイルには *.harp 拡張子を使用することをお勧めします)。

```
http://www.example.com/givememyhar.php?callback=onInputData
```

上記URLに対するレスポンスは次のようになります: 

```js
onInputData({
    "log": { ... }
});
```

live example はコチラ: http://www.softwareishard.com/har/viewer/?inputUrl=http://www.janodvarko.cz/har/viewer/examples/inline-scripts-block.harp&callback=onInputData

```html
<a href="http://www.softwareishard.com/har/viewer/?inputUrl=http://www.janodvarko.cz/har/viewer/examples/inline-scripts-block.harp&amp;callback=onInputData">http://www.softwareishard.com/har/viewer/?inputUrl=http://www.janodvarko.cz/har/viewer/examples/inline-scripts-block.harp&amp;callback=onInputData</a>
```

- `inputUrl` は対象となる HAR ファイルの URL を指定します (同じドメインからである必要はありません)。
- `callback` は、 HAR がラップされる関数の名前を指定します


## HAR Compression

HAR ファイルの圧縮は、 HAR のコア仕様の一部ではありません。
しかし、 HAR ファイルをより効率的に保存するためには、ディスク上で HAR ファイルを圧縮することが推奨されます (圧縮された HAR ファイルには *.zhar 拡張子を使用することをお勧めします)。

ただし、 HAR をサポートするアプリケーションは、圧縮された HAR ファイルをサポートする必要はありません。アプリケーションが圧縮された HAR ファイルをサポートしていない場合、 HAR ファイルをそのアプリケーションに渡す前に、ユーザーが解凍する責任があります。

HTTP 圧縮は、ウェブアプリケーションを高速化するための最善の方法の一つであり、HAR ファイルにも推奨されます。

