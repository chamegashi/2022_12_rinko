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

このコードでは、制御構造の判定に AST を用いている。

- AST: https://qiita.com/rhoboro/items/5bbaa3fa659ed7454d56

そのため、ast モジュールをインポートしている。その後、必要な自作の関数を呼び出したりしている。これはその時になったら都度説明する。

## get_cognitive_complexity

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/api.py#L9-L18

外部から呼び出すときには、この `get_cognitive_complexity` がメインとして呼ばれる。引数は`AnyFuncdef`で、返り値は `int` である。`AnyFancdef`はこのリポジトリで独自に定義されているものである。コードは以下になる。

### AnyFancdef

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/common_types.py#L1-L5

コードはいたってシンプルである。AST モジュールを呼び出し、Union 型を呼ぶ。python は非型付け言語であり、基本的には変数になんでもぶっこめる言語である。しかし、現代において多くの主流言語は型付け言語である(本来非型付け言語である JacaScript も結局 TypeScript が開発され、そちらを使う人が多い)。その流れを汲み、python 自体は非型付け言語であるが、ある程度(厳密には違うが)型を使えるようになった。それが typing であり、これを使うと引数や返り値を明示できる。

ここでは `AnyFancdef` を ast の関数 or async 関数として定義する。このどちらかを許容する型である。async 関数は非同期で動かすために必要な関数である。
- async: https://qiita.com/maueki/items/8f1e190681682ea11c98

 `get_cognitive_complexity` では冒頭で `is_decorator` が呼ばれている。

### is_decorator

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/utils/ast.py#L19-L25

`is_decorator` では python の機能であるデコレータであるかどうか判定する。Cognitive Complexity ではデコレータが使われている場合には例外的な処理がおこわなれる。デコレータでネストされた場合には、nesting が起きないという処理である。

- file: https://www.sonarsource.com/docs/CognitiveComplexity.pdf

`is_instance` 関数は python の組み込み関数である。型の一致をみる(TS だと typeof, instanceof とか)。

- 資料: https://www.javadrive.jp/python/function/index8.html

`is_decorator` が呼ばれた後は `get_cognitive_complexity` が呼ばれ、recursive になっている。

### get_cognitive_complexity_for_node
`get_cognitive_complexity`のそのあとの処理は実際に Congitive Complexity の計測になる。

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/api.py#L21-L44

引数として、

- node: ast の情報
- incremet_by: Cognitive Complexity の加算情報。初期値は0
- verbose: なんだろこれ？

を渡す。

```increment_by, base_complexity, should_iter_children = process_node_itself(node, increment_by) ```

手始めに `process_node_itself` 関数を実行している。

### process_node_itself

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/utils/ast.py#L71-L87

引数に ast と nesting のデータを受け取る。

まずは条件定義である。`control_flow_breakers` では、制御フロー中で流れが途切れてしまうような条件を tuple で定義する。`control_flow_breakers` には if, while, for, ifexp, execption がある。

`incrementers_nodes` では、Cogintive Complexity をインクリメントする条件を定義する。`incrementers_nodes` は、通常の関数、async 関数、ラムダ関数がある。

ここら辺の細かい定義には ast Document を読んだほうが早い

- Document: https://docs.python.org/ja/3/library/ast.html

https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/utils/ast.py#L88-L97

先ほど定義した `control_flow_breakers`, `incrementers_nodes` を使って、型を比較する。match する型ごとに処理が行われる。`control_flow_breakers` と一致する場合には、後述する `process_control_flow_breaker` に飛ぶ。

`incrementers_nodes` の場合には、それ以上調べることはない(関数の場合には関数内に遷移せず、計測は深堀しないのがルール)ので、return で返す。

`ast.BoolOp` では、if の中での condition の数を計測している。本来であれば、and, or の組み合わせでCogintive Complexity の値が変動するはずであれば、ここでは単に数を計測しているだけっぽい。ここからもう少し本当は手を加えたいんだろうなぁというお気持ち。

### process_control_flow_breaker
フローが途切れた場合にどういう処理を行うかの関数である。
https://github.com/chamegashi/2022_12_rinko/blob/9ef99477327ce876e6bfe576d5d52d64a7fc1242/cognitive_complexity/utils/ast.py#L45-L68

引数として、node(このときの node は `control_flow_breakers` のどれかである)と `increment_by` の２つである。ここからはそれぞれ条件で処理を分けている

- node: IfExp(else) の時

これ以上何も起きないので、単純に+1 をして返す。

- node: If で、node.orelse(elif) が一つあるとき

- node: ExceptHandler(error) の時

これ以上何も起きないので、単純に+1 をして返す。

- node: node.orelse の時

- それら以外

for, while の時なので、単純に +1 して返す

以上のような処理がされている。最終的に返す値として、`increment_by, max(1, increment_by) + increment, True` を返している。
