| フィールド                                 | 値               | 説明    |
|:--------------------------------------|:--------------------|:---------------|
| <span class="code-snippet">AffectedNodes</span>                       | 配列               | このトランザクションで作成、削除、または修正された[レジャーオブジェクト](ledger-object-types.html)のリストと、個々のオブジェクトに対する具体的な変更内容。 |
| <span class="code-snippet">DeliveredAmount</span>                     | [通貨額][] | **廃止予定。**<span class="code-snippet">delivered_amount</span>で置き換えられます。Partial Paymentsでない場合は省略されます。 |
| <span class="code-snippet">TransactionIndex</span>                    | 符号なし整数    | トランザクションが記録されているレジャーでのトランザクションの位置。この配列は0から始まります。（例えば、値が<span class="code-snippet">2</span>の場合、そのレジャーの3番目のトランザクションであったことを意味します）。 |
| <span class="code-snippet">TransactionResult</span>                   | 文字列              | トランザクションが成功したか、どのような理由で失敗したかを示す[結果コード](transaction-results.html)。 |
| [<span class="code-snippet">delivered_amount</span>](transaction-metadata.html#delivered_amount) | [通貨額][] | <span class="code-snippet">Destination</span>アカウントが実際に受取った[通貨額][]。このフィールドは、トランザクションが[Partial Payments](partial-payments.html)であるかどうかにかかわらず、送金された金額を特定するために使用します。[新規: rippled 0.27.0][] |
