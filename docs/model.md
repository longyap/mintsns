# モデル設計
TypeScript(フロント側)や、Postgres(バックエンド側)で共通して管理されるモデル論理設計です。

## Post(投稿、ポスト)
* メッセージ(本文)を持つ
* メッセージの文字制限は _N文字_ とする
* メッセージには複数の画像を添付できる
* Public投稿（一般公開）と Limited投稿（限定公開）がある
* 一般公開された投稿はインターネット上に公開される（未ログインでも閲覧可能）
* 限定公開された投稿は、投稿したタイミングで指定されたゾーン条件にマッチするユーザーが閲覧できる
* 投稿後にゾーン条件に合致したとしても過去のポストを閲覧できない
* 投稿後にゾーン条件を満たさなくなっても一度閲覧可能となった投稿は閲覧できる
* メッセージは投稿後に編集することができる
* ポストの公開設定（ゾーン条件情報など）は投稿後の編集はできない
* ポストは投稿者が任意のタイミングで削除することが出来る
* 投稿時に公開期間を設定された場合は公開期限を満了により自動的に投稿が削除（ _投稿者も閲覧不可_ ）される
* 投稿者、ポストを閲覧出来るユーザはメッセージに対するコメントを投稿できる
* ポストを閲覧出来るユーザはメッセージに対しポイントを与えることができる
* ポイントは複数種類の気持ち（絵文字で表現）のいずれかを指定して与えることができる
* タイムライン(ポストコレクション)は、InfluxDBで管理する

## Comment(コメント)
* ポストに対しコメントを投稿できる
* コメントはメッセージ（本文）を持つ
* メッセージの文字制限は _N文字_ とする
* コメントには1つだけ画像を添付可能とする
* コメントの閲覧は対象ポストを閲覧できるユーザ全てが閲覧できる
* コメントは投稿順に時系列に表示される
* コメントは投稿後に編集・削除することができる
* コメントの編集・削除は、コメントの投稿者のみが行える
* コメントは対象ポストが削除されると削除される
* コメントに対してポイントを与えることができる

## User(ユーザー)
* Google アカウントでログインする
* Google のアイコンがそのまま使用される（予定)
* キャッチフレーズが編集できる
* ユーザーに設定されてあるタグ一覧が見れる
* ユーザーのタグは、他人もしくは自分によって投票できます。

## PostGavePointUser(ポストにポイントを与えたユーザー)
* ユーザーはポストに対してポイントを与えることができる
* ひとつのポストに対して、1つだけ投票することができる
* 投票時に絵文字を使って気持ちを伝えることができる

## CommentGavePointUser(コメントにポイントを与えたユーザー)
* ユーザーはコメントに対してポイントを与えることができる
* ひとつのコメントに対して、1つだけ投票することができる
* 投票時に絵文字を使って気持ちを伝えることができる

## Notification
* 通知は100件以上の古いものは破棄する
* 通知の内容
  * コメントしたポストにコメントがありました
  * あなたのポストにコメントがありました
  * あなたの名前があります。(メンション通知)

## Stream(MainStream)
* Twitterのタイムラインのようなもの
* メインストリームは複数のZoneの条件にどれか一つでも当てはまった投稿が流れる
* Stream は ID的なものは持たず、子であるゾーン条件を使いタイムラインの中身を変化させる

## CustomStream
* ゾーンが設定されたストリームのこと
* ゾーンが設定されているので、流したい投稿を絞り込むことができる
* カスタムストリームは公開設定ができる。公開した場合は、おすすめのゾーンで紹介される。
* ここで公開されたゾーンを元に、投稿範囲をうまく調整することになる。

## Zone
* 実際にこの設定値を使ってどのようなタイムラインを配信するかは、 InfluxDB が決定する。
* ゾーンエディタがあり、ゾーンは自由に作成できる
* ゾーンは、自分のタグを見せる人を限定させる場合や、ストリームを作成する時に使用できる
* ↑<数字>と↓<数字>とxが指定できる。×はそのタグが含まれているユーザーは表示しない

## Tag
* ユーザー属性タグ
* 他人、または自分にタグをつけることができる
* タグは投票することができて、すでにあるタグに再び投票する場合は、タグポイントが加算される
* [Point] なぜ自分で自分にタグをつけられるのか
  * タグは一種類につき1ポイントしかつけることができないため、自分でタグを設定したところでは、2ポイント以上のゾーンが指定されてあるストリームを見れない。しかし、好きな趣味についてタグを設定してもらうのに他人にタグをつけてもらうようお願いすることが必ず発生してしまうし、サブアカウントで自分にタグをつけることもあるだろう。そのため、自分で自分にタグをつけるのは認める。
* タグは非公開設定できる（ただし非公開にしたタグはポイントがあがりにくくなる）
* タグにもゾーンを設定できる（あるゾーンに含まれているユーザーのみそのタグが見れる）

## MintPoint
* タグポイントの合計値のこと
* この数値の数によって人気度がわかる
