2025-08-13
アルゴリズム演習
課題1
def algorithm_01(number)
  response = []
  (1..number).each do |num|
    if (num % 15) == 0
      response << 'らんてくん'
    elsif (num % 5) == 0
      response << 'くん'
    elsif (num % 3) == 0
      response << 'らんて'
    else
      response << num
    end
  end
  response
end

課題2
def algorithm_02(word)
  word.reverse
end

課題3
def algorithm_03(word)
  odd_string = ""
  even_string = ""
  word_array = word.chars
  word_array.each_with_index do |文字, インデックス|
    if (インデックス + 1) % 2 == 1
      odd_string = odd_string + 文字
    else
      even_string = even_string + 文字
    end
  end
  odd_string + even_string
end

課題4
def algorithm_04(text)
  counts = [] 
  words = text.split
  words.each do |word|
  length = word.delete(",.").length
  counts << length
  end
  counts
end

2025-08-15
ゼロからわかるGit完全マスター講座 ～初学者からチーム開発で活躍できるエンジニアへ～

2025-8-19
プリミティブ型と文字列型をラップする
	•	ただの数値や文字列（プリミティブ型）をそのまま使うと、コード上で「この値が何を表しているのか」が分かりにくい。
	•	専用クラスでラップすると意味づけが明確になる。
例：100 ではなく Coin::ONE_HUNDRED のように扱うことで「これは金額ではなく硬貨の種類」と分かる。
	•	メリット
	•	ビジネスルールをクラスに閉じ込められる
	•	コードの可読性・保守性が上がる

ファーストクラスコレクションを使用する
	•	配列を直接使うと、意図しない操作や意味不明なコードになりやすい。
	•	用途ごとに配列をクラスでラップすることで「これは在庫」「これはお釣り」と明確にできる。

今回作成したクラス
	•	StockOf100Yen
	•	100円玉の在庫を管理する専用クラス
	•	在庫数の取得・追加・取り出しなどを一元管理
	•	Change
	•	お釣りを管理する専用クラス
	•	複数の硬貨をまとめて扱い、合計金額やクリア処理などを提供

メリット
	•	意味づけが明確になり、コードの可読性が上がる
	•	配列操作を直接書かずに、必要なメソッドだけを定義できる
	•	制御を追加しやすく、拡張に強い設計になる
（例：「100円玉は50枚まで」「お釣りは取り出し専用」など）

学びのまとめ
	•	ラップすることの意味：ただの値や配列を「意味のあるクラス」に変えることで、責務を明確にしやすくなる。
	•	短期的には冗長に見えても、長期的に保守性と拡張性を高められる。
	•	設計の耐久力を上げるリファクタリングとして有効。

2025-08-22
Getter, Setter、プロパティを使用しない
学習内容
	•	クラス内のデータを外部に直接渡さず、必要な処理だけを返すメソッドを作る
→ 内部状態を隠蔽（カプセル化）
	•	例：
	•	Stockクラス：quantity を直接参照せず empty? で在庫の有無を判定
	•	StockOf100Yen（現CashBox）: 内部配列を直接参照せず、not_have_change? でお釣り不足を判定、take_out_change でお釣りを払い出す
	•	Drinkクラス: kind を直接参照せず、coke?, diet_coke?, tea? で種類を判定

作成したクラス
	•	Stock（在庫を管理）
	•	StockOf100Yen → CashBox（100円硬貨の在庫管理）
	•	Change（お釣り管理）
	•	Drink（飲み物判定）

メリット
	•	内部状態を隠せることでカプセル化を維持
	•	オブジェクトの責務が明確になり、他クラスに影響しにくい
	•	コードの可読性・保守性が向上

学びのまとめ
	•	GetterやSetterで内部状態を直接返すのは避ける
	•	内部データを必要な処理に変換して返すことでカプセル化を守れる
	•	オブジェクト指向設計の基本的な考え方を実践できた

1つのクラスにつきインスタンス変数は2つまでにする
学習内容
	•	1クラスが複数の役割を持たないように整理
	•	VendingMachineの5つのインスタンス変数を以下の役割に分割
	1.	飲み物の在庫管理 → Storageクラス
	2.	釣り銭管理 → CoinMechクラス（CashBox + Change）

作成したクラス
	•	Storage（飲み物の在庫管理）
	•	DrinkTypeをキーにStockを保持
	•	empty?(type)、decrement(type) で操作
	•	CoinMech（釣り銭管理）
	•	CashBox（100円硬貨の在庫）とChange（お釣り）を保持
	•	硬貨投入、釣り銭払い出し、返金などを管理
	•	VendingMachine（購入処理）
	•	StorageとCoinMechに処理を委譲することで責務を整理

メリット
	•	クラスの責務が明確になり、コードがシンプル
	•	インスタンス変数が2つ以内に収まる設計
	•	拡張や保守がしやすくなる

学びのまとめ
	•	クラスを役割ごとに分割することで、複雑さを管理できる
	•	内部状態の隠蔽と責務分離の両立が可能
	•	設計の耐久力が高まるリファクタリング手法として有効
  