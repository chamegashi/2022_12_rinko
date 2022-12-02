# 説明

github に挙がっているコードの説明なので、パーマリンク使ってうまく説明できたらと思った。しかし、結構手間だったため、次回以降はやらない

# Cognitive Complexity について

前回の輪講を参照

# コード説明

まずはメインとなるコードである congitive_complexity/api を見る。

冒頭は file の import である。

https://github.com/chamegashi/2022_12_rinko/blob/9a498b625a1d9ba8946478144ae9e8e98799777c/cognitive_complexity/api.py#L1-L7

このコードでは、制御構造の判定に AST を用いている。そのため、ast モジュールをインポートしている。その後、必要な自作の関数を呼び出したりしている。これはその時になったら都度説明する。
