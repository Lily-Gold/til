RSpec入門・演習

2025-6-14
RSpecのための環境構築を行った。
ポート番号の競合によりなかなかDockerの立ち上げが上手くいかず、Gemのインストールに時間がかかった。
他にもforkする場所を気をつけるなど、環境構築は始めが肝心だと思うので次も確認を怠らないように心がけたい。

2025-6-15
モデルスペックについて学んだ。
spec/rails_helper.rbを編集し、factory botのセットアップを行った。
spec/factories/users.rbとspec/factories/tasks.rbにfactory botを利用して定義した。
spec/models/task_spec.rbにテストケースを記載した。

2025-6-16
システムスペックについて学んだ。
初めて解ける前に解答を見てしまった。
解答例の通りにやったはずなのに、エラーが出てインデントを直したらエラーが増えた。
明日もう一度挑戦し無理なら質問フォームを使うことにする。

2025-6-17
引き続きシステムスペックについて学んだ。
どうやらspec/factories/users.rbをspec/factories/user.rbと間違えて命名したせいでfactoryが上手く読み取れなかったらしい。
その後ファイル名を訂正しても上手くいかなかったが、キャッシュをクリーンすることでエラーを解決することができた。
おそらく内部ではuser.rbのデータが残っていて、そちらが参照されることによりエラーが起こっていたと思われる。

2025-6-18
RSpecの修正を行なった。
同じ操作をするとしても見やすいコードを打つということを心がけるのが大事だと思った。
内容について今日は把握きれなかったため明日もう一度確認する。
とりあえず今日はRSpecの動作に問題が発生しなかったことで終了とする。