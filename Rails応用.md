2025-07-13
Rails応用からは実務を見据えたバグ修正を中心に行う。
課題１は、記事編集画面で画像を挿入しなかった場合にプレビューもしくは公開ボタンを押すと発生するエラーの修正を行いました。
元のファイルには画像が挿入された場合しか想定されていないコードだったためエラーが起きていた。そのため上のコードを下のコードに変更。
```
.media-image
  = image_tag medium.image_url(:lg)
```
```
- if medium.image_url
  .media-image
    = image_tag medium.image_url(:lg)
```
またSlimは初めて使用したためerbとの記号の比較を書いておく。
<%= %>　→　=
<% %>　→ -
<div class="foo"> → .foo

2025-07-14
課題２ではタグ画面のみパンくずが表示されないバグを修正した。
configのbreadcrumbs.rbにパンくずが正常に表示されているカテゴリーを参考にタグのパンくず設定を行う。その際にrails routes | grep tagでルーティングを確認。以下を追記
```
crumb :admin_tags do
  link 'タグ', admin_tags_path
  parent :admin_dashboard
end

crumb :edit_admin_tag do |tag|
  link 'タグ編集', edit_admin_tag_path(tag)
  parent :admin_tags
end
```
次にコントローラーを確認するもののカテゴリーとの違いは見受けられずビューへ。
カテゴリーと比較しタグのビューではパンくずを使うための宣言が抜けていることに気がつく。
以下を追記することでパンくずの表示に成功

tags/index.html.slim
- breadcrumb :admin_tags

tags/edit.html.slim
- breadcrumb :edit_admin_tag, @tag

2025-07-17
テキスト挿入時、テキストを入力せずにプレビューページを表示するとエラーが発生するバグを修正。
sentence.body を sentence.body ||= '' に変更し、nilのときは空文字を代入するようにしてエラーを防止。

2025-07-21
記事ステータスの追加
1. 公開日時の指定を1時間単位に変更
	•	app/assets/javascripts/admin.js
	•	format: 'YYYY-MM-DD HH:mm' → format: 'YYYY-MM-DD HH:00' に変更し、1分単位から1時間単位の指定に変更。

2. 記事の状態判定を自動化
	•	app/controllers/admin/articles/publishes_controller.rb
	•	@article.state = :published → @article.state = @article.publishable? ? :published : :publish_wait に変更し、状態を条件判定によって自動設定。
	•	flash[:notice] = '記事を公開しました' → flash[:notice] = @article.message_on_published に変更し、メッセージも状態に応じて自動切り替え。


3. 記事更新時も状態を自動調整
	•	app/controllers/admin/articles_controller.rb
  @article.assign_attributes(article_params)
  @article.adjust_state
  if @article.saveに変更し、記事更新時にも状態が「公開」か「公開待ち」か自動調整されるように修正。

4. 公開待ちステータスの追加
	•	app/models/article.rb
	•	enum state に publish_wait: 2 を追加し、新しいステータス「公開待ち」を実装。
	•	以下のメソッドを追加し、状態の自動判定とメッセージ出力を実装
  def publishable?; Time.current >= published_at; end

  def message_on_published
    if published?; '記事を公開しました'
    elsif publish_wait?; '記事を公開待ちにしました'
    end
  end

  def adjust_state
    return if draft?
    self.state = publishable? ? :published : :publish_wait
  end

5. i18nの設定
	•	config/locales/enums.ja.yml
	•	publish_wait: '公開待ち' を追記し、日本語ラベルを追加。

6. cron定期実行の設定（Whenever利用）
	•	app/models/article.rb
	•	過去の公開日時を取得するスコープを追加：
  scope :past_published, -> { where('published_at <= ?', Time.current) }

	•	config/schedule.rb
	•	1時間ごとにrakeタスクが実行されるcron設定を追加：
  every :hour do
    rake 'article_state:update_article_state'
  end

	•	lib/tasks/article_state.rake
	•	公開待ち記事のステータスを自動で「公開」に変更するタスクを追加：
  namespace :article_state do
    task update_article_state: :environment do
      Article.publish_wait.past_published.find_each(&:published!)
    end
  end

2025-07-22
検索機能の追加
1. コントローラーで検索条件のパラメータを許可
app/controllers/admin/articles_controller.rb
params[:q]&.permit(:title, :category_id, :author_id, :body, :tag_id)
→ 記事一覧ページの検索で、カテゴリ・著者・タグ・記事内容・タイトルのパラメータを受け取れるように設定。

2. 検索フォームの定義（フォームオブジェクト）
app/forms/search_articles_form.rb
attribute :author_id, :integer
attribute :tag_id, :integer
attribute :body, :string
→ 著者・タグ・記事本文の検索条件をフォームオブジェクトで定義。

relation = relation.by_author(author_id) if author_id.present?
    relation = relation.by_tag(tag_id) if tag_id.present?
    body_words.each do |word|
      relation = relation.body_contain(word)
    end
→ 検索時に入力があるものだけ絞り込みクエリを実行。

def body_words
    body.present? ? body.split(nil) : []
  end
→ 記事本文の検索は、スペース区切りで複数キーワード検索に対応。

3. モデルの検索スコープ定義
app/models/article.rb
scope :by_author, ->(author_id) { where(author_id: author_id) }
scope :by_tag, ->(tag_id) { joins(:article_tags).where(article_tags: { tag_id: tag_id }) }
scope :body_contain, ->(word) { joins(:sentences).where('sentences.body LIKE ?', "%#{word}%") }
	•	by_author → 著者IDで絞り込み
	•	by_tag → 記事タグの中間テーブル経由で絞り込み
	•	body_contain → 記事の本文（sentencesテーブル）でキーワード検索

4. 検索フォームのUI実装
app/views/admin/articles/index.html.slim
 => f.select :category_id, Category.pluck(:name, :id), { include_blank: 'カテゴリ' }, class: 'form-control'
 => f.select :author_id, Author.pluck(:name, :id), { include_blank: '著者' }, class: 'form-control'
 => f.select :tag_id, Tag.pluck(:name, :id), { include_blank: 'タグ' }, class: 'form-control'
 => f.search_field :body, class: 'form-control', placeholder: '記事内容'
   = f.search_field :title, placeholder: 'タイトル', class: 'form-control'
→ 管理画面で「カテゴリ・著者・タグ・記事本文・タイトル」で複合検索できるUIを実装。

2025-07-24
アクション権限の調整
1. アクションごとのアクセス制限を追加
app/policies/taxonomy_policy.rb
def index?
  user.admin? || user.editor?
end

def update?
  user.admin? || user.editor?
end

def destroy?
  user.admin? || user.editor?
end
✔ 補足：
管理者 (admin) または 編集者 (editor) のみが
	•	一覧表示（index）
	•	編集（update）
	•	削除（destroy）
を実行できるように、ポリシーで権限を制限。

2. 認可エラー発生時の挙動を調整
config/application.rb
config.action_dispatch.rescue_responses["Pundit::NotAuthorizedError"] = :forbidden
✔ 補足：
Punditで authorize に失敗（＝許可されていない操作）すると、
403 Forbidden エラーとして処理されるように設定。
→ 従来の「500エラー（内部サーバーエラー）」のような表示を避け、適切なステータスで返す。

3. 403エラー時の画面を用意
public/403.html
<!DOCTYPE html>
<html>
<head>
  <title>権限がありません(403)</title>
  <meta name="viewport" content="width=device-width,initial-scale=1">
</head>

<body>
<p>権限がありません。</p>
</body>
</html>
✔ 補足：
ユーザーが許可されていない操作をしようとした場合に表示される静的エラーページ。
→ 403 Forbidden に対応したシンプルなUIを表示。

2025-07-26
アイキャッチの表示サイズ / 位置指定
1. 管理画面で位置・サイズを指定できるように
app/controllers/admin/articles_controller.rb
	•	Strong Parameters に以下を追加：
:eyecatch_align, :eyecatch_width
→ 記事作成・編集時にアイキャッチの**位置（左寄せ・中央・右寄せ）と横幅（100〜700px）**を指定できるようにする。

2. モデルにカラム・バリデーションを追加
app/models/article.rb
	•	カラム概要：
eyecatch_align :integer, default: 0, null: false  # left: 0, center: 1, right: 2
eyecatch_width :integer                           # 任意の横幅（100〜700）

	•	enumで位置を選択肢化：
enum eyecatch_align: { left: 0, center: 1, right: 2 }
	•	バリデーション：
validates :eyecatch_width,
          numericality: { less_than_or_equal_to: 700, greater_than_or_equal_to: 100 },
          allow_blank: true

3. 管理画面で設定できるようにビューを修正
app/views/admin/articles/edit.html.slim
	•	ラジオボタンで位置選択、数値入力で幅指定を実装：
= f.input_field :eyecatch_align, as: :radio_buttons
= f.input :eyecatch_width, placeholder: '100'

4. 記事表示側で反映
app/views/shared/_article.html.slim
	•	クラスに text-#{article.eyecatch_align} を適用し、Bootstrapなどで整列。
	•	image_tag に width: article.eyecatch_width を渡して任意幅指定。
section class="eye_catch text-#{article.eyecatch_align}"
  = image_tag article.eye_catch_url(:lg), class: 'img-fluid', width: article.eyecatch_width

5. i18n対応
config/locales/activerecord.ja.yml
eyecatch_align: '位置'
eyecatch_width: '横幅'

config/locales/enums.ja.yml
eyecatch_align:
  left: '左寄せ'
  center: '中央'
  right: '右寄せ'

6. マイグレーションファイル
db/migrate/×××××_add_eye_catch_info_to_articles.rb
class AddEyeCatchInfoToArticles < ActiveRecord::Migration[7.0]
  def change
    add_column :articles, :eyecatch_align, :integer, default: 0, null: false
    add_column :articles, :eyecatch_width, :integer
  end
end

db/schema.rb
t.integer "eyecatch_align", default: 0, null: false
t.integer "eyecatch_width"

2025-07-27
埋め込みメディアタイプにTwitterの追加
1. 埋め込み種別の追加
app/models/embed.rb
enum embed_type: { youtube: 0, twitter: 1 }
	•	Embed モデルで、埋め込みのタイプを識別するために enum を使用。
	•	youtube? / twitter? で埋め込みタイプを条件分岐できるようになります。

2. YouTube埋め込みIDの整形
def split_id_from_youtube_url
    # YoutubeならIDのみ抽出
    identifier.split('/').last if youtube?
  end
	•	以前は identifier に YouTube動画の IDだけ を直接保存していたが、今後はURL形式で保存。
	•	このメソッドで、URLからID部分（例: "https://youtu.be/abc123" → "abc123"）を抽出

3. 表示アイコンの切り替え
app/decorators/article_block_decorator.rb
blockable.youtube? ? '<i class="fa fa-youtube-play"></i>'.html_safe : '<i class="fa fa-twitter"></i>'.html_safe
	•	記事ブロックの装飾（UI）で、YouTubeとTwitterのアイコンを出し分け。

4. ブロック挿入画面でのUI
app/views/admin/articles/article_blocks/_insert_block.html.slim
 .d-inline-flex
            i.fa.fa-youtube-play
            i.fa.fa-twitter
	•	管理画面でのブロック挿入UIに、Twitterブロックの選択肢を追加。

5. 埋め込み表示の切り替え
app/views/admin/articles/article_blocks/_show_embed.html.slim
- if embed.identifier?
    - if embed.youtube?

    - if embed.twitter?
      = render 'shared/embed_twitter', embed: embed
	•	記事に埋め込まれた動画・投稿を表示する際に、タイプに応じて部分テンプレートを切り替え。

6. 表示用テンプレートの追加
Twitter用: app/views/shared/_embed_twitter.html.slim
script async="" charset="utf-8" src="https://platform.twitter.com/widgets.js"
.embed-twitter
  blockquote.twitter-tweet
    a href="#{embed.identifier}"

YouTube用: app/views/shared/_embed_youtube.html.slim
= content_tag 'iframe', nil, width: width, height: height, src: "https://www.youtube.com/embed/#{embed.split_id_from_youtube_url}", \
	•	Twitter は blockquote + TwitterウィジェットJSを使って埋め込み表示
  •	YouTube は <iframe> で埋め込み

7. 表示ラベルの日本語化
config/locales/enums.ja.yml
twitter: 'Twitter'
	•	enumの表示に対応するラベルを追加（管理画面などで「Twitter」と日本語で表示）

8. データ移行用Rakeタスク
lib/tasks/temp/fix_embed_youtube_identifier.rake
namespace :fix_embed_youtube_identifier do
  desc 'IDを入力していたidentiferカラムの過去データを一括で修正'
  task update_old_identifier_for_youtube_embed: :environment do
    Embed.youtube.each do |embed|
      embed.update(identifier: "https://youtu.be/#{embed.identifier}")
    end
  end
end
	•	過去データの修正用タスク。
	•	以前は "abc123" のようにIDだけを保存していたが、今後は "https://youtu.be/abc123" のようにURL形式で保存。
	•	このタスクは全 youtube タイプの identifier に "https://youtu.be/" をプレフィックスとして付け直すもの。
