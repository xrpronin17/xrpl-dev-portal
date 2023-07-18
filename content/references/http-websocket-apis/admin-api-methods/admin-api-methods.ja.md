---
html: admin-api-methods.html
parent: http-websocket-apis.html
blurb: これらの管理APIメソッドを使用してrippledサーバーを管理します。
labels:
  - コアサーバー
---
# 管理APIメソッド

<span class="code-snippet">rippled</span>サーバーと直接通信する際には管理APIメソッドを使用します。管理メソッドは、信頼できるサーバー運用担当者のみを対象としています。管理メソッドには、サーバーの管理、監視、デバッグのためのコマンドが含まれています。

管理コマンドを使用できるのは、管理者として、<span class="code-snippet">rippled.cfg</span>ファイルに指定されているホストとポートで<span class="code-snippet">rippled</span>サーバーに接続している場合に限られます。デフォルトでは、コマンドラインクライアントが管理接続を使用します。<span class="code-snippet">rippled</span>への接続についての詳細は、[rippled API入門](get-started-using-http-websocket-apis.html)を参照してください。


## [キー生成メソッド](key-generation-methods.html)

キーを生成および管理するには、以下のメソッドを使用します。

* **[<span class="code-snippet">validation_create</span>](validation_create.html)** - 新しいrippledバリデータのキーを生成します。
* **[<span class="code-snippet">wallet_propose</span>](wallet_propose.html)** - 新規アカウントのキーを生成します。


## [ロギングおよびデータ管理のメソッド](logging-and-data-management-methods.html)

ログレベルとその他のデータ（レジャーなど）の管理には、以下のメソッドを使用します。

* **[<span class="code-snippet">can_delete</span>](can_delete.html)** - 特定レジャーまでのレジャーのオンライン削除を許可します。
* **[<span class="code-snippet">download_shard</span>](download_shard.html)** - レジャー履歴の特定のシャードをダウンロードします。
* **[<span class="code-snippet">ledger_cleaner</span>](ledger_cleaner.html)** - レジャークリーナーサービスが破損データを確認するように設定します。
* **[<span class="code-snippet">ledger_request</span>](ledger_request.html)** - ピアサーバーに対し特定のレジャーバージョンを照会します。
* **[<span class="code-snippet">log_level</span>](log_level.html)** - ログの詳細レベルを取得または変更します。
* **[<span class="code-snippet">logrotate</span>](logrotate.html)** - ログファイルを再度開きます。


## [サーバー制御メソッド](server-control-methods.html)

rippledサーバーの管理には、以下のメソッドを使用します。

* **[<span class="code-snippet">connect</span>](connect.html)** - rippledサーバーを特定のピアに強制的に接続します。
* **[<span class="code-snippet">ledger_accept</span>](ledger_accept.html)** - スタンドアロンモードでレジャーを閉鎖し、次のレジャーに進みます。
* **[<span class="code-snippet">stop</span>](stop.html)** - rippledサーバーをシャットダウンします。
* **[<span class="code-snippet">validation_seed</span>](validation_seed.html)** - 検証に使用するキーを一時的に設定します。


## [ステータスおよびデバッグメソッド](status-and-debugging-methods.html)

ネットワークとサーバーのステータスを確認するには、以下のメソッドを使用します。

* **[<span class="code-snippet">consensus_info</span>](consensus_info.html)** - 発生したコンセンサスの状態に関する情報を取得します。
* **[<span class="code-snippet">feature</span>](feature.html)** - プロトコルAmendmentに関する情報を取得します。
* **[<span class="code-snippet">fetch_info</span>](fetch_info.html)** - サーバーとネットワークの同期に関する情報を取得します。
* **[<span class="code-snippet">get_counts</span>](get_counts.html)** - サーバー内部とメモリー使用状況に関する統計情報を取得します。
* **[<span class="code-snippet">peers</span>](peers.html)** - 接続しているピアサーバーに関する情報を取得します。
* **[<span class="code-snippet">print</span>](print.html)** - 内部サブシステムに関する情報を取得します。
* **[<span class="code-snippet">validators</span>](validators.html)** - 現在のバリデータに関する情報を取得します。
* **[<span class="code-snippet">validator_list_sites</span>](validator_list_sites.html)** - バリデータリストを公開するサイトに関する情報を取得します。


## 廃止予定のメソッド

以下の管理コマンドは廃止予定であり、今後予告なしに削除される可能性があります。

* <span class="code-snippet">ledger_header</span> - 代わりに[ledgerメソッド][]を使用してください。
* <span class="code-snippet">unl_add</span>、<span class="code-snippet">unl_delete</span>、<span class="code-snippet">unl_list</span>、<span class="code-snippet">unl_load</span>、<span class="code-snippet">unl_network</span>、<span class="code-snippet">unl_reset</span>、<span class="code-snippet">unl_score</span> - 代わりに UNL管理用構成ファイルを使用してください。
* <span class="code-snippet">wallet_seed</span> - 代わりに[wallet_proposeメソッド][]を使用してください。


<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}			
{% include '_snippets/tx-type-links.md' %}			
{% include '_snippets/rippled_versions.md' %}
