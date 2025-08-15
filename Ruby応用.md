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