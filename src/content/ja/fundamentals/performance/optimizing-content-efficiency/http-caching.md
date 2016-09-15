project_path: /web/_project.yaml
book_path: /web/fundamentals/_book.yaml
description: ネットワーク経由で情報を取得することは時間もコストもかかります。レスポンスが大きいと、クライアントとサーバー間のラウンドトリップを何度も繰り返す必要があるため、レスポンスが利用可能となってブラウザで処理できるようになるまで時間がかかり、またユーザー側ではデータの通信コストも発生します。そのため、以前取得したリソースをキャッシュに格納して再使用できるようにしておくことは、パフォーマンスを最適化する上で重要な要素です。

{# wf_updated_on: 2014-01-04 #}
{# wf_published_on: 2013-12-31 #}

# HTTP キャッシュの作成 {: .page-title }

{% include "web/_shared/contributors/ilyagrigorik.html" %}



ネットワーク経由で情報を取得することは時間もコストもかかります。レスポンスが大きいと、クライアントとサーバー間のラウンドトリップを何度も繰り返す必要があるため、レスポンスが利用可能となってブラウザで処理できるようになるまで時間がかかり、またユーザー側ではデータの通信コストも発生します。そのため、以前取得したリソースをキャッシュに格納して再使用できるようにしておくことは、パフォーマンスを最適化する上で重要な要素です。



うれしいことに、どのブラウザにも HTTP キャッシュは実装されています。したがって、各サーバーのレスポンスに正しい HTTP ヘッダー ディレクティブがあり、ブラウザがレスポンスをキャッシュに格納できるタイミングとその期間をブラウザに指示していることを確認するだけで済みます。

Note: Webview を使用してウェブ コンテンツを取得しアプリケーションに表示する場合は、構成フラグを追加することで、HTTP キャッシュが有効であること、用途に合わせてキャッシュのサイズが適切な値に設定されていること、キャッシュが存続していることを確認する必要があります。プラットフォームの資料で設定を確認してください。

<img src="images/http-request.png" class="center" alt="HTTP リクエスト">

サーバーがレスポンスを返したときに、コンテンツ タイプ、長さ、キャッシュ ディレクティブ、検証トークンなどが記述された一連の HTTP ヘッダーも出力されます。上記の例では、サーバーは 1024 バイトのレスポンスを返し、クライアントにそのレスポンスを最大 120 秒間キャッシュに格納するよう指示して、検証トークン（"x234dff"）を提供します。この検証トークンを使って、レスポンスの有効期限が切れた後、リソースに変更があったかどうかを確認できます。


## ETag によるキャッシュ内のレスポンスの検証

## TL;DR {: .hide-from-toc }
- サーバーから ETag HTTP ヘッダーを介して検証トークンが渡される。
- 検証トークンを使うことで効率的なリソース更新チェックが可能となる。リソースに変更がなければデータ転送は発生しない。


初回の取得から 120 秒経過し、ブラウザが同じリソースに対する新しいリクエストを開始したとしましょう。まず、ブラウザはローカル キャッシュを調べて前回のレスポンスを見つけます。残念ながら、レスポンスは「有効期限切れ」のため使用できません。この時点で、ブラウザは新しいリクエストを発行して新しい完全なレスポンスを取得することもできますが、この処理は非効率的です。リソースに変更がなければ、キャッシュ内のデータとまったく同じものをダウンロードする必要はありません。

これは、ETag ヘッダーで指定された検証トークンで解決できる問題です。サーバーは任意のトークンを生成して返します。通常、このトークンはファイルのコンテンツのハッシュやその他のフィンガープリントです。クライアントはフィンガープリントがどう生成されるかを認識する必要はありません。必要なのは次回のリクエストでそのフィンガープリントをサーバーに送信することだけです。フィンガープリントが変わらなければリソースに変更はないので、ダウンロードを省略できます。

<img src="images/http-cache-control.png" class="center" alt="HTTP Cache-Control の例">

上記の例では、クライアントは "If-None-Match" HTTP リクエスト ヘッダー内で ETag トークンを自動的に提供し、サーバーはそのトークンを現在のリソースと突き合わせて確認します。リソースに変更がなければ、"304 Not Modified" レスポンスを返し、キャッシュ内のレスポンスに変更がないことをブラウザに通知し、さらに 120 秒後に更新を延期できます。レスポンスを再度ダウンロードする必要がないため、時間と帯域幅を節約できます。

ウェブ デベロッパーとして、効率のよい再検証をどう利用したらよいでしょうか。ブラウザがすべて処理してくれます。ブラウザは検証トークンが以前に指定されたかどうかを自動的に検出し、発信リクエストにそのトークンを付加し、サーバーから受信したレスポンスに基づき必要に応じてキャッシュのタイムスタンプを更新します。**あとは、サーバーが、実際に、必要な ETag トークンを提供していることを確認することだけです。必要な構成フラグについては、サーバーの資料をご確認ください。**

Note: おすすめの方法: HTML5 Boilerplate プロジェクトには、人気の高いすべてのサーバー向けの<a href='https://github.com/h5bp/server-configs'>サンプル構成ファイル</a>が用意され、構成フラグと設定ごとに詳細なコメントが記載されています。リストでご希望のサーバーを見つけ、該当する設定を探してコピーするか、推奨される設定でサーバーが設定されていることを確認してください。


## Cache-Control

## TL;DR {: .hide-from-toc }
- リソースごとに Cache-Control HTTP ヘッダーでキャッシュ ポリシーを設定できる
- Cache-Control ディレクティブで、レスポンスのキャッシュを許可するユーザー、キャッシュの条件、キャッシュに格納する期間を制御する


最良のリクエストとは、サーバーに伝える必要がないリクエストです。レスポンスのローカルコピーがあれば、ネットワークによる待ち時間はまったく発生せず、データ転送のためにデータを読み込む必要もありません。これを実現するため、HTTP 仕様では、ブラウザのキャッシュやその他の中間キャッシュに各レスポンスを格納できる条件やその期間を制御する[さまざまな Cache-Control ディレクティブ](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9)をサーバーが返すことを許可しています。

Note: Cache-Control ヘッダーは HTTP/1.1 仕様の一部として定義されたもので、レスポンス キャッシュ ポリシーの設定に使用されていた以前のヘッダー（Expires など）に優先します。最新のブラウザはすべて Cache-Control に対応しているので、必要なのは指定することだけです。

<img src="images/http-cache-control-highlight.png" class="center" alt="HTTP Cache-Control の例">

### "no-cache" と "no-store"

「no-cache」は、返されたレスポンスを使って同じ URL に対する後続のリクエストに応えるには、レスポンスに変更があったかどうかをまずサーバーで確認する必要があることを示します。そのため、適切な検証トークン（ETag）がある場合に no-cache が指定されていると、キャッシュ内のレスポンスを検証するためのラウンドトリップは発生しますが、レスポンスに変更がなければダウンロードを省略できます。

逆に「no-store」はもっと単純です。返されたレスポンスのバージョンにかかわらず、ブラウザのキャッシュやすべての中間キャッシュはそのレスポンスを一切格納できません。たとえば、個人の機密データや銀行データが含まれているレスポンスなどです。ユーザーがこのアセットをリクエストするたびに、リクエストがサーバーに送信され、毎回、完全なレスポンスがダウンロードされます。

### "public" と "private"

レスポンスが "public" として指定されている場合は、レスポンスに関連付けられた HTTP 認証があり、さらにレスポンスのステータス コードが通常キャッシュ可能になっていない場合でも、そのレスポンスはキャッシュに格納できます。ほとんどの場合、明示的なキャッシュ情報（"max-age" など）で
レスポンスがキャッシュ可能であることが指定されているので、"public" は必要ありません。

逆に "private" レスポンスは、ブラウザのキャッシュには格納できますが、通常、対象ユーザーは 1 人のため、中間キャッシュに格納することは認められません。たとえば、個人的なユーザー情報を含む HTML ページはそのユーザーのブラウザでのみキャッシュに格納でき、CDN では格納できません。

### "max-age"

このディレクティブでは、取得したレスポンスを再使用できる最大時間を、リクエストの時刻を起点とする秒数で指定します。たとえば、"max-age=60" は、レスポンスを 60 秒間キャッシュに格納して再使用できることを示します。

## 最適な Cache-Control ポリシーの設定

<img src="images/http-cache-decision-tree.png" class="center" alt="キャッシュ決定ツリー">

上記の決定ツリーに沿って、特定のリソース、またはアプリケーションで使用する一連のリソースに最適なキャッシュ ポリシーを決定してください。理想的には、できる限り多くのレスポンスを、できる限り長期間、クライアントのキャッシュに格納し、レスポンスごとに検証トークンを提供して再検証を効率化することをおすすめします。

<table>
<thead>
  <tr>
    <th width="30%">Cache-Control ディレクティブ</th>
    <th>説明</th>
  </tr>
</thead>
<tr>
  <td data-th="cache-control">max-age=86400</td>
  <td data-th="説明">レスポンスは、最大 1 日（60 秒 x 60 分 x 24 時間）、ブラウザのキャッシュやすべての中間キャッシュに格納できる（つまり、"public" である）</td>
</tr>
<tr>
  <td data-th="cache-control">private, max-age=600</td>
  <td data-th="説明">レスポンスは、最大 10 分（60 秒 x 10 分）、クライアントのブラウザのみがキャッシュに格納できる</td>
</tr>
<tr>
  <td data-th="cache-control">no-store</td>
  <td data-th="説明">レスポンスはキャッシュに格納することが認められていないため、リクエストごとに完全に取得する必要がある。</td>
</tr>
</table>

HTTP Archive によると、上位 300,000 件のサイト（Alexa ランク順）の間では、[ダウンロードした全レスポンスのほぼ半数をブラウザのキャッシュに格納できます](http://httparchive.org/trends.php#maxage0)。つまり、繰り返しの多いページビューやアクセスでは大幅なコスト削減となります。もちろん、これは、特定のアプリケーションでキャッシュに格納できるのはリソースの 50% である、という意味ではありません。リソースの 90% 以上をキャッシュに格納できるサイトもあれば、キャッシュに一切格納できない個人的なデータや時間依存のデータが多いサイトもあります。

**ページを監査してどのリソースをキャシュに格納できるかを見極め、そのリソースが適切な Cache-Control ヘッダーと ETag ヘッダーを返していることを確認してください。**


## キャッシュ内のレスポンスの無効化と更新

## TL;DR {: .hide-from-toc }
- ローカル キャッシュに格納されたレスポンスはリソースの「有効期限」まで使用される
- ファイル コンテンツのフィンガープリントを URL に埋め込むことで、クライアントが新しいバージョンのレスポンスに更新するよう設定できる
- パフォーマンスを最適化するにはアプリケーションごとに独自のキャッシュ階層を設定する必要がある


ブラウザからの HTTP リクエストはすべて、まずブラウザのキャッシュに転送され、キャッシュ内にリクエストを満たす有効なレスポンスがあるかどうかを確認します。一致するレスポンスがあると、そのレスポンスがキャッシュから読み取られるため、ネットワークによる待ち時間も、転送によるデータの通信コストも発生しません。**では、キャッシュ内のレスポンスを更新したり無効にしたりするにはどうすればよいのでしょうか。**

たとえば、CSS スタイルシートを最大 24 時間（max-age=86400）キャッシュに格納することを指定しましたが、デザイナーがすべてのユーザーに利用してほしい更新をコミットしたとしましょう。どれが CSS の「古い」キャッシュ コピーであるかをユーザーに伝え、ユーザーにキャッシュを更新してもらうにはどうするのでしょうか。これは微妙な質問です。このためには、少なくともリソースの URL を変更することが必要になります。

レスポンスをブラウザがキャッシュに格納すると、そのキャッシュ バージョンは、max-age や expires で指定されたとおりに有効期限が切れるまで、または他の何らかの理由でキャッシュから削除される（たとえば、ユーザーがブラウザのキャッシュを消去する）まで、使用されます。そのため、ページの作成時に使用されるファイルのバージョンがユーザーごとに異なることがあります。リソースを取得して間もないユーザーは新しいバージョンを使用することになりますが、キャッシュに以前の（ただし有効な）コピーがあるユーザーは古いバージョンのレスポンスを使用することになります。

**では、クライアント側のキャッシュとクイック アップデートの両方のメリットを活用するにはどうすればよいでしょうか。**簡単です。リソースの URL を変更し、コンテンツが変わるたびにユーザーに新しいレスポンスをダウンロードしてもらえばよいのです。通常、この処理はファイルのフィンガープリント、またはバージョン番号をファイル名に埋め込む（たとえば、style.**x234dff**.css など）で実現されます。

<img src="images/http-cache-hierarchy.png" class="center" alt="キャッシュ階層">

リソース単位でキャッシュ ポリシーを設定できれば「キャッシュ階層」を設定でき、各リソースをキャッシュに格納する期間だけでなく、ユーザーが新しいバージョンを確認する間隔も制御できるようになります。たとえば、上記の例を分析してみましょう。

* HTML では「no-cache」が指定されているので、ブラウザはリクエストごとに必ずドキュメントを再検証し、コンテンツに変更がある場合に最新のバージョンを取得します。また、HTML マークアップ内には CSS アセットと JavaScript アセットの URL にフィンガープリントも埋め込まれています。ファイルのコンテンツに変更があると、ページの HTML も変更され、HTML レスポンスの新しいコピーがダウンロードされます。
* CSS はブラウザのキャッシュと中間キャッシュ（たとえば、CDN）への格納が認められ、有効期限が 1 年に設定されています。1 年という「かなり先の有効期限」を安心して使用できるのは、ファイルのフィンガープリントがファイル名に埋め込まれているためです。CSS が更新されると、URL も変更されます。
* JavaScript の有効期限も 1 年に設定されていますが、private とし指定されています。これは、CDN がキャッシュに格納できない個人的なユーザーデータが含まれているためです。
* 画像のキャッシュにはバージョンも一意のフィンガープリントも使用されません。有効期限は 1 日に設定されます。

ETag、Cache-Control、一意の URL を組み合わせることで、長い有効期限、レスポンスのキャッシュが可能な場所の制御、オンデマンドの更新といった希望どおりの結果を実現することができます。

## キャッシュ チェックリスト

最善のキャッシュ ポリシーなど 1 つとしてありません。トラフィックのパターン、提供するデータの種類、データの更新に対するアプリケーション別の要件に応じて、リソースごとに適切な設定を行い、全体的な「キャッシュ階層」を構成する必要があります。

次に、キャッシュ戦略に取り組む際に覚えておくべきおすすめの方法を紹介します。

1. **一貫性のある URL を使用する:** 同じコンテンツを異なる URL で提供すると、そのコンテンツの取得と格納は何度も繰り返されます。おすすめの方法: [URL は大文字と小文字が区別されます](http://www.w3.org/TR/WD-html40-970708/htmlweb.html)。
2. **サーバーが検証トークン（ETag）を提供していることを確認する:** サーバーでリソースに変更がない場合、検証トークンがあれば同じデータを転送する必要がなくなります。
3. **中間でキャッシュに格納できるリソースを指定する:** すべてのユーザー間でレスポンスが変わらないリソースが、CDN などの中間でのキャッシュ候補です。
4. **リソースごとに最適なキャッシュ有効期限を決定する:** リソースによって更新要件は異なります。リソースごとに適切な max-age を監査して決定します。
5. **サイトに最適なキャッシュ階層を決定する:** リソースの URL をコンテンツのフィンガープリントと組み合わせ、HTML ドキュメントの有効期限を短縮するか no-cache を指定することで、クライアントが更新を取得する間隔を制御できます。
6. **チャーンを最小限に抑える:** 他より更新頻度が高いリソースもあります。リソースの更新頻度が部分的に高い場合は（JavaScript 関数、一連の CSS スタイルなど）、そのコードを別ファイルとして提供することを検討してください。別ファイルにしておけば、コンテンツの残りの部分（変更頻度が低いライブラリ コードなど）はキャッシュから取得できるようになり、更新を取得するたびにダウンロードするコンテンツの容量を最小限に抑えることができます。



