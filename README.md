# 説明

github に挙がっているコードの説明なので、パーマリンク使ってうまく説明できたらと思った。しかし、結構手間だったため、次回以降はやらない

# Cognitive Complexity について

前回の輪講を参照

URL: http://sp.cei.uec.ac.jp/internal/wiki/lab/pukiwiki/index.php?%C5%C4%C3%E6%28%B2%ED%29%2FCognitive%20Complexity

# コード説明

まずはメインとなるコードである `congitive_complexity/api` を見る。

### import

冒頭は file の import である。

https://github.com/chamegashi/2022_12_rinko/blob/9a498b625a1d9ba8946478144ae9e8e98799777c/cognitive_complexity/api.py#L1-L7

このコードでは、制御構造の判定に AST を用いている。そのため、ast モジュールをインポートしている。その後、必要な自作の関数を呼び出したりしている。これはその時になったら都度説明する。

### get_cognitive_complexity

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/api.py#L9-L18

外部から呼び出すときには、この `get_cognitive_complexity` がメインとして呼ばれる。引数は`AnyFuncdef`である。これはこのリポジトリで独自に定義されているものである。コードは以下になる。

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/common_types.py#L1-L5

コードはいたってシンプルである。AST モジュールを呼び出し、Union 型を呼ぶ。python は非型付け言語であり、基本的には変数になんでもぶっこめる言語である。しかし、現代において多くの主流言語は型付け言語である(本来非型付け言語である JacaScript も結局 TypeScript が開発され、そちらを使う人が多い)。その流れを汲み、python 自体は非型付け言語であるが、ある程度(厳密には違うが)型を使えるようになった。それが typing であり、これを使うと引数や返り値を明示できる。

ここでは `AnyFancdef` を ast の関数 or async 関数として定義する。このどちらかを許容する型である。async 関数は非同期で動かすために必要な関数である。
https://qiita.com/maueki/items/8f1e190681682ea11c98

 `get_cognitive_complexity` では冒頭で `is_decorator` が呼ばれている。

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/utils/ast.py#L19-L25

`is_decorator` では python の機能であるデコレータであるかどうか判定する。
