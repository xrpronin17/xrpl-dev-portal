<span class="code-snippet">rippled</span>が残りのネットワークと同期されるまでには数分かかることがあります。その間、レジャーがない旨を知らせる警告が出力されます。

<span class="code-snippet">rippled</span>ログメッセージの詳細は、[ログメッセージについて](understanding-log-messages.html)を参照してください。

<span class="code-snippet">rippled</span>が残りのネットワークと同期されたら、ストック<span class="code-snippet">rippled</span>サーバーが完全に機能するようになります。このサーバーを、ローカル署名やXRP LedgerへのAPIアクセスに使用できます。<span class="code-snippet">rippled</span>サーバーがネットワークと同期されているかどうかを判別するには、[<span class="code-snippet">rippled</span>サーバーの状況](rippled-server-states.html)を使用します。[<span class="code-snippet">rippled</span>のコマンドラインインターフェイス](get-started-using-http-websocket-apis.html#コマンドライン)を使用すれば、これを迅速にテストできます。

{% if currentpage.md == "tutorials/manage-the-rippled-server/installation/build-run-rippled-ubuntu.md" or
      currentpage.md == "tutorials/manage-the-rippled-server/installation/build-run-rippled-macos.md" %}
    $ ./rippled server_info
{% else %}
    $ /opt/ripple/bin/rippled server_info
{% endif %}

rippled APIを使用した<span class="code-snippet">rippled</span>サーバーとの通信について詳しくは、[rippled API reference](http-websocket-apis.html)を参照してください。

ストック<span class="code-snippet">rippled</span>サーバーを実行できたら、次に検証サーバーとして実行してみましょう。検証サーバーについて、そして検証サーバーを実行する理由については、[バリデータとしてのrippledの実行](run-rippled-as-a-validator.html)を参照してください。

<span class="code-snippet">rippled</span>サーバーの起動でお困りですか? [rippledサーバーが起動しない](server-wont-start.html)を参照してください。

### その他の構成

<span class="code-snippet">rippled</span>は、デフォルト構成でXRP Ledgerに接続する必要があります。ただし、<span class="code-snippet">rippled.cfg</span>ファイルを編集すれば、設定を変更できます。推奨される構成設定については、[容量の計画](capacity-planning.html)を参照してください。

{% include '_snippets/conf-file-location.md' %}<!--_ -->

すべての構成オプションの説明については、[<span class="code-snippet">rippled</span> GitHubリポジトリー](https://github.com/ripple/rippled/blob/master/cfg/rippled-example.cfg)を参照してください。

構成の変更を有効にするには、<span class="code-snippet">rippled</span>を再起動する必要があります。

{% if currentpage.md == "tutorials/manage-the-rippled-server/installation/install-rippled-on-ubuntu.md" or
      currentpage.md == "tutorials/manage-the-rippled-server/installation/install-rippled-on-centos-rhel-with-yum" %}
        $ sudo systemctl restart rippled.service

{% elif currentpage.md == "tutorials/manage-the-rippled-server/installation/build-run-rippled-ubuntu.md" or
        currentpage.md == "tutorials/manage-the-rippled-server/installation/build-run-rippled-macos.md" %}


* Ctrl-Cを使用して<span class="code-snippet">rippled</span>を停止し、その後再起動します。

      $ ./rippled

{% endif %}

<span class="code-snippet">[debug_logfile]</span>セクションまたは<span class="code-snippet">[database_path]</span>セクションを変更すると、<span class="code-snippet">rippled</span>を実行するユーザーに、新しく構成したパスの所有権を付与する必要が生じる場合があります。

### 更新

<span class="code-snippet">rippled</span>を定期的に更新して、残りのXRP Ledgerネットワークと同期させておく必要があります。[rippledのGoogleグループ](https://groups.google.com/forum/#!forum/ripple-server)をサブスクライブすれば、<span class="code-snippet">rippled</span>の新しいリリースに関する通知を受け取ることができます。

<span class="code-snippet">rippled</span>のパッケージには、[Linuxでの自動更新を有効にする](update-rippled-automatically-on-linux.html)ために使用できるスクリプトが含まれています。その他のプラットフォームでは、手動での更新が必要です。
