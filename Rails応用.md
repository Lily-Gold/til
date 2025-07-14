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