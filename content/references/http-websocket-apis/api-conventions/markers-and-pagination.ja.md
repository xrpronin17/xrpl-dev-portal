---
html: markers-and-pagination.html
parent: api-conventions.html
blurb: 大きなクエリを複数の応答にページネーションする際の慣例です。
---
# マーカーとページネーション

一部のメソッドから返されるデータは、1つの応答に実質的に収まらないことがあります。結果全体が収まらない場合、応答には<span class="code-snippet">marker</span>フィールドが含まれます。このフィールドを使用することで、複数回の呼出しを通じてデータのページをさらに取得できます。各要求で直前の応答の<span class="code-snippet">marker</span>値を渡して、終わったところから再開します。応答に<span class="code-snippet">marker</span>が含まれていなければ、データセットの終わりに達しています。

<span class="code-snippet">marker</span>フィールドのフォーマットは意図的に未定義になっています。各サーバーで<span class="code-snippet">marker</span>をそれぞれに合わせて定義できるので、このフィールドの形式は文字列、ネストオブジェクトなどさまざまです。異なるサーバー、または同じサーバーの異なるメソッドでは、異なる<span class="code-snippet">marker</span>定義を使用できます。各<span class="code-snippet">marker</span>は一時的であり、10分以上経過すると予期されているとおりに機能しなくなることがあります。
