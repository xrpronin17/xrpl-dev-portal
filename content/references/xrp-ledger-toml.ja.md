---
html: xrp-ledger-toml.html
parent: references.html
blurb: 機械が読み取れる、あなたに関する情報を他のXRP Ledgerユーザーに提供します。
curated_anchors:
    - name: ファイルの提供方法
      anchor: "#ファイルの提供方法"
    - name: 内容
      anchor: "#内容"
    - name: CORSの設定
      anchor: "#corsの設定"
    - name: ドメイン検証
      anchor: "#ドメイン検証"
    - name: アカウント検証
      anchor: "#アカウント検証"
---
# xrp-ledger.tomlファイル

XRP Ledgerバリデータを実行するか、ビジネスにXRP Ledgerを使用する場合、機械が読み取れる形式の **<span class="code-snippet">xrp-ledger.toml</span>** ファイルで自分のXRP Ledgerの使用状況に関する情報を世界中に提供することができます。スクリプトとアプリケーションで<span class="code-snippet">xrp-ledger.toml</span>ファイルの情報を使用して、XRP Ledgerでの自分の情報を分かりやすく示すことができます。また、同じファイルを人間が読むことが便利な場合もあります。

<span class="code-snippet">xrp-ledger.toml</span>ファイルの主な使用例のひとつに、[ドメインの検証](#ドメイン検証)があります。

#### 表記法

本書のキーワード「MUST」、「MUST NOT」、「REQUIRED」、「SHALL」、「SHALL NOT」、「SHOULD」、「SHOULD NOT」、「RECOMMENDED」、「MAY」、および「OPTIONAL」は、[RFC 2119](https://tools.ietf.org/html/rfc2119)の記述どおりに解釈されます。

## ファイルの提供方法

<span class="code-snippet">xrp-ledger.toml</span>ファイルは、Webサーバーより提供することを前提としています。このファイルは、次のURLで提供できます。

```
https://{DOMAIN}/.well-known/xrp-ledger.toml
```

<span class="code-snippet">{DOMAIN}</span>には、お使いのドメイン名が入ります（サブドメインを含む）。例えば、次のいずれのURLからでもファイルを提供できます。

```
https://example.com/.well-known/xrp-ledger.toml
https://xrp.services.example.com/.well-known/xrp-ledger.toml
```

### プロトコル

上記の内容は、セキュリティ上、**HTTPSプロトコル**経由で提供する必要があります。その際、[TLS](https://tools.ietf.org/html/rfc8446)の最新の安全なバージョンと、既知の認証局によって署名された有効な証明書を使用します。（注記: TLSは以前はSSLと呼ばれていましたが、現在それらのバージョンは安全とは言えません。）つまり、<span class="code-snippet">xrp-ledger.toml</span>ファイルをホストするための前提条件は、適切に構成されたHTTPSのWebサーバーを用意することです。

プレーンHTTPプロトコルは中間者攻撃に対して脆弱です。例えば、一部のインターネットサービスは、プレーンHTTP経由で取得したコンテンツを、独自の広告が挿入されるように改変していることが知られています。同様の技術によって<span class="code-snippet">xrp-ledger.toml</span>ファイルの内容が誤って伝達され、スクリプトの誤動作または虚偽的な動作につながる可能性を防ぐため、プレーンHTTP経由で提供される<span class="code-snippet">xrp-ledger.toml</span>ファイルの内容を信頼すべきではありません。

### ドメイン

<span class="code-snippet">xrp-ledger.toml</span>ファイルを提供するドメインは、所有権の表明とも言えます。このようなファイルの内容は、単独で提供されている場合、有用または信頼できるものではありません。実用的な観点からすれば、メインドメインからファイルを提供することは望ましくない場合もあります。そのため、任意の数のサブドメインを使用することができます。[XRP Ledgerアカウントの<span class="code-snippet">Domain</span>フィールド](accountset.html#domain)を設定するときは、使用したすべてのサブドメインを含むフルドメインを指定する必要があります。詳細は、[アカウントの検証](#アカウント検証)を確認してください。

必要に応じて、複数のサブドメインから同じファイルを提供することもできます。例えば、サブドメイン<span class="code-snippet">www.example.com</span>から<span class="code-snippet">example.com</span>と同じWebサイトに遷移する場合は、どちらからもファイルを提供できます。Webサイトでプレフィクス<span class="code-snippet">www</span>が必要な場合は、ドメインを指定する際（例えば、XRP Ledgerアカウントの<span class="code-snippet">Domain</span>フィールドを設定する際）に必ずプレフィクスを含めてください。

<span class="code-snippet">xrp-ledger.toml</span>ファイルと同じドメインから人間が読める形式のWebサイトを提供することをお勧めします。このWebサイトでは、あなたのIDおよびXRP Ledgerの使用方法に関する詳細情報を提供します。この情報は、あなたやあなたのサービスに対する信頼を構築するのに役立ちます。

### パス

[RFC5785](https://tools.ietf.org/html/rfc5785)に準拠し、パスの開始部分は<span class="code-snippet">/.well-known/</span>でなければなりません。ファイルは、パス<span class="code-snippet">/.well-known/xrp-ledger.toml</span>で正しく表示される必要があります（すべて小文字）。

必要に応じて、同じファイルを、大文字と小文字を変えて（例: <span class="code-snippet">/.well-known/XRP-Ledger.TOML</span>）提供することもできます。パスが大文字か小文字かによって、異なるコンテンツを提供することはできません。

### ヘッダー

#### Content-Type

<span class="code-snippet">xrp-ledger.toml</span>ファイルに推奨される**Content-Type**は<span class="code-snippet">application/toml</span>です。ただし、このファイルを使用するアプリケーションは、Content-Typeの値として<span class="code-snippet">text/plain</span>も受け入れる必要があります。

#### CORS

他のWebサイト上のスクリプトでファイルを照会できるようにするには、ファイルに対して[CORS][]を有効にする必要があります。具体的には、サーバーで<span class="code-snippet">xrp-ledger.toml</span>を提供するときに次のヘッダーを付加する必要があります。

```
Access-Control-Allow-Origin: *
```

このヘッダーが付加されるようにサーバーを構成する方法については、[CORSセットアップ](#corsの設定)を参照してください。

#### その他のヘッダー

サーバーでは、必要に応じて、圧縮、キャッシュ制御、リダイレクト、関連リソースのリンクなど、他の標準HTTPヘッダーを使用できます。

### 生成

<span class="code-snippet">xrp-ledger.toml</span>ファイルは、Webサーバーに格納されている実際のファイルである場合もあれば、Webサーバーによってオンデマンドで生成される場合もあります。ファイルで提供されるコンテンツまたはWebサイトの構成によっては、後者の方が望ましい場合があります。

## 内容

<span class="code-snippet">xrp-ledger.toml</span>ファイルの内容は、[TOML](https://github.com/toml-lang/toml)でフォーマットする必要があります。 **内容はすべてオプションです。** コメントは省略できますが、分かりやすくするために推奨されます。

サンプルコンテンツ:

```
# Example xrp-ledger.toml file. These contents should not be considered
# authoritative for any real entity or business.
# Note: all fields and all sections are optional.

[METADATA]
modified = 2019-01-22T00:00:00.000Z
expires = 2019-03-01T00:00:00.000Z

[[VALIDATORS]]
public_key = "nHBtDzdRDykxiuv7uSMPTcGexNm879RUUz5GW4h1qgjbtyvWZ1LE"
network = "main"
owner_country = "us"
server_country = "us"
unl = "https://vl.ripple.com"

[[VALIDATORS]]
public_key = "nHB57Sey9QgaB8CubTPvMZLkLAzfJzNMWBCCiDRgazWJujRdnz13"
network = "testnet"
owner_country = "us"
server_country = "us"
unl = "https://vl.testnet.rippletest.net"

[[ACCOUNTS]]
address = "r3kmLJN5D28dHuH8vZNUZpMC43pEHpaocV"
desc = "Ripple-owned address from old ripple.txt file"
# Note: This doesn't prove ownership of an account unless the
#   "Domain" field of the account in the XRP Ledger matches the
#   domain this file was served from.

[[SERVERS]]
json_rpc = "https://s1.ripple.com:51234/"
ws = "wss://s1.ripple.com/"
peer = "https://s1.ripple.com:51235/"
desc = "General purpose server cluster"

[[SERVERS]]
json_rpc = "https://s2.ripple.com:51234/"
ws = "wss://s2.ripple.com/"
peer = "https://s2.ripple.com:51235/"
desc = "Full-history server cluster"

[[SERVERS]]
json_rpc = "https://s.testnet.rippletest.net:51234/"
ws = "wss://s.testnet.rippletest.net:51233/"
peer = "https://s.testnet.rippletest.net:51235/"
network = "testnet"
desc = "Test Net public server cluster"

[[PRINCIPALS]]
name = "Rome Reginelli" # Primary spec author
email = "rome@example.com" # Not my real email address

[[CURRENCIES]]
code = "LOL"
issuer = "rf1BiGeXwwQoi8Z2ueFYTEXSwuJYfV2Jpn"
network = "testnet"
display_decimals = 2
symbol = "😆" # In practical situations, it may be unwise to use emoji

# End of file
```

### Metadata

メタデータセクションには、<span class="code-snippet">xrp-ledger.toml</span>ファイル自体に関する情報が表示されます。このセクションは、（存在する場合） _単一_ 角括弧を使用した<span class="code-snippet">[METADATA]</span>行で開始する単一のテーブルでなければなりません。（<span class="code-snippet">xrp-ledger.toml</span>ファイルの他のほとんどのセクションでは、情報の配列のために二重括弧を使用していますが、<span class="code-snippet">[METADATA]</span>セクションは最大で1つしかありません。）以下から任意のフィールドを指定できます（大文字と小文字は区別されます）。

| フィールド      | 型             | 説明                                  |
|:-----------|:-----------------|:---------------------------------------------|
| <span class="code-snippet">modified</span> | オフセット日時 | <span class="code-snippet">xrp-leder.toml</span>ファイルが最後に変更された時間。    |
| <span class="code-snippet">expires</span>  | オフセット日時 | 現在の時間がこの時間と同じか以降である場合、<span class="code-snippet">xrp-ledger.toml</span>ファイルは期限切れと見なされます。 |

本仕様では<span class="code-snippet">domain</span>フィールドは定義していません。このフィールドは、ファイルを提供するサイトから決定する必要があります。

**ヒント:** Rippleでは、オフセット日時の値についてオフセット<span class="code-snippet">Z</span>を使用し、ミリ秒単位まで精度を高めることを推奨しています。（例:  <span class="code-snippet">2019-01-22T22:26:58.027Z</span>）ファイルを手動で編集する場合は、時間、分、秒、ミリ秒にゼロを指定して、時間を概算で示すことができます。(例: <span class="code-snippet">2019-01-22T00:00:00.000Z</span>)

### バリデータ

バリデータリストには、操作するサーバーの検証に関する情報が表示されます。バリデータリストは、（存在する場合）各エントリが二重角括弧のヘッダー<span class="code-snippet">[[VALIDATORS]]</span>を使用する複数のテーブルの配列でなければなりません。各エントリで、別個の検証サーバーが示されます。

ファイル内の _最初の_<span class="code-snippet">[[VALIDATORS]]</span>エントリは、プライマリのバリデータとして扱われます。本番環境のXRP Ledgerで1つ以上のバリデータを実行する場合は、他のバリデータに信頼されるバリデータを最初に配置する必要があります。

_各_<span class="code-snippet">[[VALIDATORS]]</span>エントリごとに、以下から任意のフィールドを指定することができます。

| フィールド        | 型   | 説明                                          |
|:-------------|:-------|:-----------------------------------------------------|
| <span class="code-snippet">public_key</span> | 文字列 | XRP Ledgerのbase58形式でエンコードされた、プライマリバリデータのマスター公開鍵（通常、先頭は<span class="code-snippet">n</span>）。 |
| <span class="code-snippet">network</span>  | 文字列 | このバリデータがフォローするネットワークチェーン。省略された場合、クライアントはバリデータが本番環境のXRP Ledgerをフォローすると想定する必要があります。本番環境のXRP Ledgerを明示的に指定するには<span class="code-snippet">main</span>を使用します。RippleのXRP Ledger Test Netには<span class="code-snippet">testnet</span>を使用します。他の値を指定して、他のTest Netまたは非標準のネットワークチェーンを表すことができます。
| <span class="code-snippet">owner_country</span> | 文字列 | ISO-3166-2形式の国コード（2桁）。あなた（バリデータの所有者）が属する主な法的管轄域を表します。 |
| <span class="code-snippet">server_country</span> | 文字列 | ISO-3166-2形式の国コード（2桁）。この検証サーバーがある物理的な場所を表します。 |
| <span class="code-snippet">unl</span> | 文字列 | HTTPS URL。このバリデータが信頼する、他のバリデータのリストを確認できます。バリデータがUNL推奨のバリデータリストサイトを使用するように構成されている場合は、サーバーの構成と一致しなければなりません。本番環境のXRP Ledgerネットワークの場合は、<span class="code-snippet">https://vl.ripple.com</span>（末尾のスラッシュは省略可）を使用します。 |

### アカウント

アカウントリストには、所有するXRP Ledgerアカウントに関する情報が表示されます。アカウントリストは、（存在する場合）各エントリが二重角括弧を含んだヘッダー<span class="code-snippet">[[ACCOUNTS]]</span>を使用する複数のテーブルの配列でなければなりません。各エントリで、個別のアカウントが示されます。 _各_<span class="code-snippet">[[ACCOUNTS]]</span>エントリごとに、以下から任意のフィールドを指定することができます。

| フィールド     | 型   | 説明                                             |
|:----------|:-------|:--------------------------------------------------------|
| <span class="code-snippet">address</span> | 文字列 | XRP Ledgerのbase58フォーマットでエンコードされた、アカウントの公開アドレス（通常、先頭は<span class="code-snippet">r</span>）。 |
| <span class="code-snippet">network</span> | 文字列 | このアカウントが主に使用されるネットワークチェーン。省略された場合、クライアントはアカウントが本番環境のXRP Ledger _と_ 、場合によっては他のネットワークチェーンでも要求されていると想定する必要があります。本番環境のXRP Ledgerでは<span class="code-snippet">main</span>を使用します。RippleのXRP Ledger Test Netには<span class="code-snippet">testnet</span>を使用します。他の値を指定して、他のTest Netまたは非標準のネットワークチェーンを表すことができます。 |
| <span class="code-snippet">desc</span> | 文字列 | 人間が読める形式での、このアカウントの目的または使用方法に関する説明。 |

**注意:** <span class="code-snippet">xrp-ledger.toml</span>ファイルをホストすることで、誰でもアカウントの所有権を主張できるため、ここでのアカウントの存在が正式であると見なすべきではありません。ただし、[XRP Ledgerの当該アカウントの<span class="code-snippet">Domain</span>フィールド](accountset.html#domain)が、この<span class="code-snippet">xrp-ledger.toml</span>ファイルの提供元のドメインとも一致している場合は除きます。詳細は、[アカウントの検証](#アカウント検証)を確認してください。

### 責任者

責任者リストには、XRP Ledgerのビジネスおよびサービスに関係するユーザー（または企業）に関する情報が表示されます。責任者リストは、（存在する場合）各エントリが二重角括弧を含んだヘッダー<span class="code-snippet">[[PRINCIPALS]]</span>を使用する複数のテーブルの配列でなければなりません。各エントリは異なる連絡先を示します。 _各_<span class="code-snippet">[[PRINCIPALS]]</span>エントリごとに、以下から任意のフィールドを指定することができます。

| フィールド   | 型   | 説明                                              |
|:--------|:-------|:-----------------------------------------------------|
| <span class="code-snippet">name</span>  | 文字列 | この責任者の名前。                                    |
| <span class="code-snippet">email</span> | 文字列 | この責任者の連絡先メールアドレス。                     |

必要に応じて、他の連絡先情報を指定することができます。（カスタムフィールドについては、[カスタムフィールド](#カスタムフィールド)を参照してください。）

### サーバー

サーバーリストには、パブリックアクセスで実行するXRP Ledgerサーバー（<span class="code-snippet">rippled</span>）に関する情報が表示されます。サーバーリストは、（存在する場合）各エントリが二重角括弧を含んだヘッダー<span class="code-snippet">[[SERVERS]]</span>を使用する複数のテーブルの配列でなければなりません。各エントリは、異なるサーバーまたはサーバークラスターを示します。 _各_<span class="code-snippet">[[SERVERS]]</span>エントリごとに、以下から任意のフィールドを指定することができます。

| フィールド   | 型   | 説明                                              |
|:--------|:-------|:-----------------------------------------------------|
| <span class="code-snippet">json_rpc</span> | 文字列（URL） | 公開JSON-RPC APIを提供するURL。先頭は<span class="code-snippet">http://</span>または<span class="code-snippet">https://</span>である必要があります。公開APIに対してはHTTPSが推奨されます。 |
| <span class="code-snippet">ws</span> | 文字列（URL） | 公開WebSocket APIを提供するURL。先頭は<span class="code-snippet">ws://</span>または<span class="code-snippet">wss://</span>である必要があります。公開APIに対してはWSSが推奨されます。 |
| <span class="code-snippet">peer</span> | 文字列（URL） | サーバーがXRP LedgerのピアプロトコルをリッスンしているURL。他のXRP Ledgerサーバーは、このURLで接続できます。サーバーよりピアクローラー応答が提供される場合は、このURLに<span class="code-snippet">crawl</span>が付加されて提供されます。 |
| <span class="code-snippet">network</span>  | 文字列 | このサーバーがフォローするネットワークチェーン。省略された場合、クライアントはサーバーが本番環境のXRP Ledgerをフォローすると想定する必要があります。本番環境のXRP Ledgerを明示的に指定するには<span class="code-snippet">main</span>を使用します。RippleのXRP Ledger Test Netには<span class="code-snippet">testnet</span>を使用します。他の値を指定して、他のTest Netまたは非標準のネットワークチェーンを表すことができます。 |

このセクションのすべてのURLの末尾にスラッシュを付けることが推奨されます。省略された場合、クライアントアプリケーションでは、末尾にスラッシュが含まれていると想定します。

### 通貨

XRP Ledgerで資産、トークン、または通貨を発行する場合は、<span class="code-snippet">[[CURRENCIES]]</span>リストでそれらに関する情報を提供できます。通貨リストは、（存在する場合）各エントリが二重角括弧を含んだヘッダー<span class="code-snippet">[[CURRENCIES]]</span>を使用する複数のテーブルの配列でなければなりません。各エントリは、個別の発行済み通貨または資産を示します。 _各_<span class="code-snippet">[[CURRENCIES]]</span>エントリごと _に_ 、以下から任意のフィールドを指定することができます。

| フィールド   | 型   | 説明                                              |
|:--------|:-------|:------------------------------------------------------|
| <span class="code-snippet">code</span> | 文字列 | XRP Ledgerでのこの通貨の（大文字と小文字を区別する）ティッカーシンボル。例えば、3桁のコード、40文字の16進コード、またはカスタム形式（XRP Ledgerで非標準コードの表記を把握しているクライアント用）があります。XRP Ledgerの通貨コードフォーマットの詳細は、[通貨コードリファレンス](currency-formats.html#通貨コード)を参照してください。 |
| <span class="code-snippet">display_decimals</span> | 数値 | クライアントアプリケーションがこの通貨の金額を表示するために使用する十進数。 |
| <span class="code-snippet">issuer</span> | 文字列 | この通貨を発行するXRP Ledgerアカウントのアドレス。XRP Ledgerのbase58形式でエンコードされます（通常、先頭は<span class="code-snippet">r</span>）。また、このアドレスは<span class="code-snippet">[[ACCOUNTS]]</span>リストに表示する必要があります。（留意事項: ここにアドレスが存在することは、それ自体では権限を持ちません。詳細は、[アカウントの検証](#アカウント検証)を参照してください。) |
| <span class="code-snippet">network</span> | 文字列 | この通貨を発行するネットワークチェーン。本番環境のXRP Ledgerを明示的に指定するには<span class="code-snippet">main</span>を使用します。省略された場合、クライアントは通貨が本番環境のXRP Ledgerで発行されていると想定する必要があります。RippleのXRP Ledger Test Netには<span class="code-snippet">testnet</span>を使用します。他の値を指定して、他のTest Netまたは非標準のネットワークチェーンを表すことができます。 |
| <span class="code-snippet">symbol</span> | 文字列 | 「$」や「€」などのテキスト記号。Unicode規格の記号がある場合に、資産または通貨の金額で使用されます。 |

### カスタムフィールド

<span class="code-snippet">xrp-ledger.toml</span>ファイルは、XRP Ledgerのユーザーが、他のユーザーやスクリプト、アプリケーションに情報を提供することを目的としています。そのため、内容を伝えるのに役立つものの、この仕様には記載されていない多くの種類の情報が存在する可能性があります。ユーザーは、必要に応じて関連情報を伝えるために、<span class="code-snippet">xrp-ledger.toml</span>ファイルの任意のレベルで他のフィールドを追加することをお勧めします。

<span class="code-snippet">xrp-ledger.toml</span>ファイルを解析するツールは、アプリケーションが把握していないその他のフィールドを含むドキュメントを受け入れる必要があります。そのようなツールは、これらの追加フィールドを呼び出す上位レベルのアプリケーションで、追加フィールドを使用できるようにするか、それらのフィールドを無視することができます。この仕様の将来のバージョンとのフォワードコンパティビリティを維持するために、ツールはこの標準で指定されたフィールドを無視することもできます。<span class="code-snippet">xrp-ledger.toml</span>ファイルに認識されていないフィールドが含まれている場合、ツールはエラーを返すべきではありません。タイプミスを検出するために、ツールは認識されていないフィールド（特に、フィールド名が標準フィールド名と類似している場合など）に警告を表示することができます。

認識したフィールドが予期されたとおりのフォーマットではない場合、そのフィールドがこの仕様で定義されていなくても、ツールがエラーを返す場合もあります。

カスタムフィールドを作成するときは、選択するフィールド名に注意してください。ごく一般的なフィールド名を使用すると、他のユーザーが同じ名前で異なるものを意味するか、競合する方法でフォーマットされる場合があります。他のユーザーにとって有益だと思われるカスタムフィールドを使用する場合は、このドキュメントのメンテナーにそのフィールドの仕様を提供してください。

## CORSの設定

<span class="code-snippet">xrp-ledger.toml</span>ファイルに対してCross-Origin Resource Sharing（[CORS][]）を許可するようにWebサーバーを構成する必要があります。この構成は、Webサーバーによって異なります。

Apache HTTP Serverを実行する場合は、以下の内容を構成ファイルに追加します。

```
<Location "/.well-known/xrp-ledger.toml">
    Header set Access-Control-Allow-Origin "*"
</Location>
```

または、以下の内容をサーバーの<span class="code-snippet">/.well-known/</span>ディレクトリ内の<span class="code-snippet">.htaccess</span>ファイルに追加できます。

```
<Files "xrp-ledger.toml">
    Header set Access-Control-Allow-Origin "*"
</Files>
```

nginxを使用する場合は、以下の内容を構成ファイルに追加します。

```
location /.well-known/xrp-ledger.toml {
    add_header 'Access-Control-Allow-Origin' '*';
}
```

他のWebサーバーについては、[サーバーにCORSサポートを追加したい](https://enable-cors.org/server.html)を参照してください。マネージドホスティングを使用する場合、特定のパスでCORSを有効にする方法については、Webホストのドキュメントを参照してください。（Webサイト全体でCORSを有効にすることはあまりないと思われます。）

[CORS]: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS

## ドメイン検証

<span class="code-snippet">xrp-ledger.toml</span>ファイルの用途のひとつは、同じエンティティが特定のドメインを運用し、かつバリデータの公開鍵によって識別される特定のバリデータも運用することを検証することです。ドメインとバリデータが同じエンティティによって所有されていることを検証することで、バリデータオペレーターとしてのIDを確実に保証できます。また、これは信頼できるバリデータになるための推奨手順でもあります。（他の推奨事項については、[優れたバリデータの特徴の理解](run-rippled-as-a-validator.html#1-優れたバリデータの特徴の理解)を参照してください。）

ドメイン検証では、ドメインオペレーターとバリデータの間に双方向リンクを確立する必要があります。

1. ドメインはバリデータの所有権を主張します。

   - [本書に記載されているすべての要件](#ファイルの提供方法)に従って、当該ドメインから<span class="code-snippet">xrp-ledger.toml</span>ファイルを提供します。

   - その<span class="code-snippet">xrp-ledger.toml</span>ファイルで、<span class="code-snippet">[[VALIDATORS]]</span>エントリの<span class="code-snippet">public_key</span>フィールドにバリデータのマスター公開鍵を入力します。

2. バリデータは、ドメイン別に所有権を主張します。このステップの手順は、本仕様書の範囲外です。

    <!-- TODO: Link documentation when it's available. -->
    <!-- Aside: the old way of doing this was to set the `Domain` field of the
         XRP Ledger account whose public key matched the validator's public key.
         Since only the holder of the corresponding private key could send
         transactions from this address, doing so effectively proved ownership
         of the validator's private key.
    -->

## アカウント検証

[ドメイン検証](#ドメイン検証)と同様に、アカウント検証は、同じエンティティが特定のドメインと特定のXRP Ledgerアドレスを運用していることを証明するという考え方を示します。XRP Ledgerの使用や<span class="code-snippet">xrp-ledger.toml</span>ファイルの提供にはアカウント検証は必要ありませんが、透明性の確保のために、アカウントを検証することもできます。

アカウント検証では、ドメインオペレーターとアドレスの間に双方向リンクを確立する必要があります。

1. ドメインはアドレスの所有権を主張します。

   - [本書に記載されているすべての要件](#ファイルの提供方法)に従って、当該ドメインから<span class="code-snippet">xrp-ledger.toml</span>ファイルを提供します。

   - その<span class="code-snippet">xrp-ledger.toml</span>ファイルで、検証するアカウントのアドレスを<span class="code-snippet">[[ACCOUNTS]]</span>エントリに入力します。このアドレスから通貨を発行する場合は、<span class="code-snippet">[[CURRENCIES]]</span>エントリの<span class="code-snippet">issuer</span>フィールドでこのアカウントを指定することもできます。

2. アドレスは、ドメイン別に所有権を主張します。

   この<span class="code-snippet">xrp-ledger.toml</span>ファイルの提供元のドメインに一致するように、[アカウントの<span class="code-snippet">Domain</span>フィールドを設定](accountset.html#domain)します。ドメイン値（ASCIIからデコードされた場合）は、<span class="code-snippet">www.</span>などのすべてのサブドメインを含め、 _正確に_ 一致する必要があります。国際化ドメイン名の場合は、[RFC3492](https://tools.ietf.org/html/rfc3492)の説明に従って、<span class="code-snippet">Domain</span>値をドメインのPunycodeに設定します。

   <span class="code-snippet">Domain</span>を設定するにはトランザクションを送信する必要があるため、<span class="code-snippet">Domain</span>値を設定する場合は必ずトランザクションを送信した時のアカウントの秘密鍵を所有している必要があります。

これらの2つのリンクのどちらも、それ自体で、権限があると見なされるべきではありません。誰でも任意のアカウントの所有権を主張する<span class="code-snippet">xrp-ledger.toml</span>ファイルをホストでき、任意のアカウントオペレーターはその<span class="code-snippet">Domain</span>フィールドを任意の文字列に設定できます。両者が一致したら、いずれも同じエンティティが操作しているという強力な証拠になります。

## 謝辞

この仕様は、[オリジナルのripple.txt仕様](https://web.archive.org/web/20161007113240/https://wiki.ripple.com/Ripple.txt)をもとに作成されたものであり、[stellar.tomlファイル](https://www.stellar.org/developers/guides/walkthroughs/how-to-complete-stellar-toml.html)から発想を得たものです。この仕様には、XRPコミュニティメンバーと、過去および現在の多くのRipple従業員からのフィードバックも組み込まれています。
