math
----

算術式を評価する。

.. code-block:: cmake

  math(EXPR <variable> "<expression>" [OUTPUT_FORMAT <format>])

算術式の ``<expression>`` を評価し、その結果を ``<variable>`` に格納します。
``<expression>`` の評価結果は 64ビットの符号付き整数として表現できるようにして下さい。
浮動小数点数を含む算術式は無効です（例えば ``1.1 * 10``）。
整数ではない結果は切り捨てられます（例えば ``3 / 2`` の評価結果）。

``<expression>`` は一個の文字列として、二重引用符で囲んで指定して下さい（例えば ``"5 * (10 + 13)"``）。
サポートしている演算子は ``+``、``-``、``*``、``/``、``%``、``|``、``&``、``^``、``~``、``<<``、``>>``、``(...)`` です（これら意味はＣ言語の演算子と同じ）。

.. versionadded:: 3.13
  Ｃ言語のように、先頭に ``0x`` を付けると16進数として認識されるようになった。

.. versionadded:: 3.13
  評価結果は ``OUTPUT_FORMAT`` オプションに指定した ``<format>`` で整形されるようになった。``<format>`` には次のいずれかを指定すること：

  ``HEXADECIMAL``
    16進数表記（先頭の文字が "``0x``"）にする。
  ``DECIMAL``
    10進数表記にする（これが ``OUTPUT_FORMAT`` オプションを指定しなかった場合のデフォルト）。

このコマンドの例：

.. code-block:: cmake

  math(EXPR value "100 * 0xA" OUTPUT_FORMAT DECIMAL)      # value には "1000" が格納される
  math(EXPR value "100 * 0xA" OUTPUT_FORMAT HEXADECIMAL)  # value には "0x3e8" が格納される
