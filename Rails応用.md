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
