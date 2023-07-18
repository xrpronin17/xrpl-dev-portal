---
html: feature.html
parent: status-and-debugging-methods.html
blurb: Amendmentに関してこのサーバーが認識している情報を返します。
labels:
  - ブロックチェーン
  - コアサーバー
---
# feature
[[ソース]](https://github.com/ripple/rippled/blob/master/src/ripple/rpc/handlers/Feature1.cpp "Source")

<span class="code-snippet">feature</span>コマンドは、[Amendment](amendments.html)に関してこのサーバーが認識している情報（Amendmentが有効であるかどうか、サーバーが[Amendmentプロセス](amendments.html#amendmentプロセス)でこれらのAmendmentに賛成票を投じたかどうかなど）を返します。[新規: rippled 0.31.0][]

<span class="code-snippet">feature</span>コマンドを使用して、Amendmentへの賛成票または反対票を投じるようにサーバーを一時的に設定できます。この変更は、サーバーの再起動後までは持続しません。Amendment投票で持続する変更を行うには<span class="code-snippet">rippled.cfg</span>ファイルを使用します。詳細は、[Amendment投票の設定](amendments.html#amendment投票の設定)を参照してください。

_<span class="code-snippet">feature</span>メソッドは、権限のないユーザーは実行できない[管理メソッド](admin-api-methods.html)です。_

### 要求フォーマット
要求フォーマットの例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket - すべてリスト*

```json
{
 "id": "list_all_features",
 "command": "feature"
}
```

*WebSocket - 拒否*

```json
{
 "id": "reject_multi_sign",
 "command": "feature",
 "feature": "4C97EBA926031A7CF7D7B36FDE3ED66DDA5421192D63DE53FFB46E43B9DC8373",
 "vetoed": true
}
```

*JSON-RPC*

```json
{
   "method": "feature",
   "params": [
       {
           "feature": "4C97EBA926031A7CF7D7B36FDE3ED66DDA5421192D63DE53FFB46E43B9DC8373",
           "vetoed": false
       }
   ]
}
```

*コマンドライン*

```sh
#Syntax: feature [<feature_id> [accept|reject]]
rippled feature 4C97EBA926031A7CF7D7B36FDE3ED66DDA5421192D63DE53FFB46E43B9DC8373 accept
```

<!-- MULTICODE_BLOCK_END -->

要求には以下のパラメーターが含まれます。

| `Field`   | 型    | 説明                                            |
|:----------|:--------|:-------------------------------------------------------|
| <span class="code-snippet">feature</span> | 文字列  | _（省略可）_ Amendmentの一意のID（16進数）またはAmendmentの短い名前。指定されている場合は、応答が1つのAmendmentに限定されます。それ以外の場合は応答にすべてのAmendmentのリストが表示されます。 |
| <span class="code-snippet">vetoed</span>  | ブール値 | （省略可、<span class="code-snippet">feature</span>が指定されていない場合は無視されます）trueの場合、サーバーに対し<span class="code-snippet">feature</span>で指定されたAmendmentに反対票を投じるように指示します。falseの場合、サーバーに対しAmendmentに賛成票を投じるように指示します。 |

**注記:** サーバーが新しいAmendmentの適用方法を現在認識していない場合でも、<span class="code-snippet">feature</span>フィールドにAmendment IDを指定すれば、新しいAmendmentに賛成票を投じるようにサーバーを設定できます。たとえば、Amendmentを _確かに_ サポートする新しい<span class="code-snippet">rippled</span>バージョンに間もなくアップグレードする予定がある場合などにこのように設定できます。

### 応答フォーマット

処理が成功した応答の例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket - すべてリスト*

```json
{
 "id": "list_all_features",
 "status": "success",
 "type": "response",
 "result": {
   "features": {
     "42426C4D4F1009EE67080A9B7965B44656D7714D104A72F9B4369F97ABF044EE": {
       "enabled": false,
       "name": "FeeEscalation",
       "supported": true,
       "vetoed": false
     },
     "4C97EBA926031A7CF7D7B36FDE3ED66DDA5421192D63DE53FFB46E43B9DC8373": {
       "enabled": false,
       "name": "MultiSign",
       "supported": true,
       "vetoed": false
     },
     "6781F8368C4771B83E8B821D88F580202BCB4228075297B19E4FDC5233F1EFDC": {
       "enabled": false,
       "name": "TrustSetAuth",
       "supported": true,
       "vetoed": false
     },
     "C1B8D934087225F509BEB5A8EC24447854713EE447D277F69545ABFA0E0FD490": {
       "enabled": false,
       "name": "Tickets",
       "supported": true,
       "vetoed": false
     },
     "DA1BD556B42D85EA9C84066D028D355B52416734D3283F85E216EA5DA6DB7E13": {
       "enabled": false,
       "name": "SusPay",
       "supported": true,
       "vetoed": false
     }
   }
 }
}
```

*WebSocket - 拒否*

```json
{
   "id": "reject_multi_sign",
   "status": "success",
   "type": "response",
   "result": {
       "features": {
           "4C97EBA926031A7CF7D7B36FDE3ED66DDA5421192D63DE53FFB46E43B9DC8373": {
               "enabled": false,
               "name": "MultiSign",
               "supported": true,
               "vetoed": true
           }
       }
   }
}
```

*JSON-RPC*

```json
200 OK

{
   "result": {
       "4C97EBA926031A7CF7D7B36FDE3ED66DDA5421192D63DE53FFB46E43B9DC8373": {
           "enabled": false,
           "name": "MultiSign",
           "supported": true,
           "vetoed": false
       },
       "status": "success"
   }
}
```

*コマンドライン*

```json
Loading: "/etc/rippled.cfg"
Connecting to 127.0.0.1:5005

{
   "result": {
       "4C97EBA926031A7CF7D7B36FDE3ED66DDA5421192D63DE53FFB46E43B9DC8373": {
           "enabled": false,
           "name": "MultiSign",
           "supported": true,
           "vetoed": false
       },
       "status": "success"
   }
}
```

<!-- MULTICODE_BLOCK_END -->

応答は[標準フォーマット][]に従っており、正常に完了した場合は結果に**Amendmentのマップ**がJSONプロジェクトとして含まれています。オブジェクトのキーはAmendment IDです。各キーの値は、そのIDのAmendmentのステータスを記述した _Amendmentオブジェクト_ です。要求に<span class="code-snippet">feature</span>が指定されいる場合、要求による変更の適用後には、要求されたAmendmentオブジェクトだけがマップに含まれます。各Amendmentオブジェクトのフィールドを次に示します。

| `Field`     | 型    | 説明                                          |
|:------------|:--------|:-----------------------------------------------------|
| <span class="code-snippet">enabled</span>   | ブール値 | 最新レジャーでこのAmendmentが現在有効であるかどうか。 |
| <span class="code-snippet">name</span>      | 文字列  | （省略される場合があります）このAmendmentの人間が読める形式の名前（判明している場合）。 |
| <span class="code-snippet">supported</span> | ブール値 | サーバーがこのAmendmentの適用方法を認識しているかどうか。このフィールドが<span class="code-snippet">false</span>（サーバーがこのAmendmentの適用方法を認識していない）に設定されており、<span class="code-snippet">enabled</span>が<span class="code-snippet">true</span>（このAmendmentが最新レジャーで有効である）に設定されている場合、このAmendmentによりサーバーが[Amendment blocked](amendments.html#amendment-blocked)になる可能性があります。 |
| <span class="code-snippet">vetoed</span>    | ブール値 | サーバーがこのAmendmentに反対票を投じるように指示されているかどうか。 |

**注意:** Amendmentの<span class="code-snippet">name</span>は、Amendmentの内容を厳密に示すものではありません。サーバー間でこの名前が一意であることや整合性があることは保証されません。

### 考えられるエラー

* [汎用エラータイプ][]のすべて。
* <span class="code-snippet">badFeature</span> - 指定されている<span class="code-snippet">feature</span>のフォーマットが正しくないか、サーバーがその名前のAmendmentを認識していません。

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
