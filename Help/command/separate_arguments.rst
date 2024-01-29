separate_arguments
------------------

コマンドライン引数を解析して :ref:`セミコロンで区切られたリスト <CMake Language Lists>` に格納する。

.. code-block:: cmake

  separate_arguments(<variable> <mode> [PROGRAM [SEPARATE_ARGS]] <args>)

空白で区切られた文字列 ``<args>`` をパースして要素に分解し、それらを「:ref:`セミコロンで区切られたリスト <CMake Language Lists>`」の ``<variable>`` に格納します。

このコマンドの目的はコマンドライン引数を分解することです。
コマンドライン全体を一つの文字列として ``<args>`` に渡して下さい。

``<args>`` をパースする方法は CMake を実行するホストのプラットフォームによって異なります。
パースする方法は ``<mode>`` オプションに指定します。
``<mode>`` には、以下のいずれかを指定して下さい：

``UNIX_COMMAND``
  ``<args>`` は空白（引用符なし）で区切られている文字列とする。
  引用符（``'`` と ``"`` ）は要素として扱われる。
  バックスラッシュ（``\`` ）は、それに続くリテラル（文字）をエスケープする（例えば ``\"`` は ``"`` ）。
  エスケープに特殊なケースは無い（例えば ``\n`` は改行文字ではなく、ただの ``n`` ）。

``WINDOWS_COMMAND``
  Windows でプログラムを起動した時に生成される argv と同じ構文で ``<args>`` をパースする。
  ``<args>`` は空白（二重引用符なし）で区切られている文字列とする。
  バックスラッシュ（``\`` ）は、それに続くのが二重引用符（``"`` ）でない限り、リテラル（文字）をエスケープする（例えば ``\'`` は ``'`` ）。
  詳細は MSDN のドキュメント「`Parsing C Command-Line Arguments`_」を参照のこと。

``NATIVE_COMMAND``
  .. versionadded:: 3.9

  ホストのプラットフォームが Windows の場合は ``WINDOWS_COMMAND`` が指定されたものとして処理する。
  それ以外のプラットフォームの場合は ``UNIX_COMMAND`` が指定されたものとして処理する。

``PROGRAM``
  .. versionadded:: 3.19

  ``<args`` の先頭の要素を実行形式とみなし、システムの検索パスで検索するか、または絶対パスなら検索しないでそのまま使う。
  検索パスの中から見つからなかったら ``<variable>`` は空である。
  それ以外は ``<variable>`` は次の要素からなるリストになる：

    0. 実行形式の絶対パス
    1. ``<args>`` の残り部分（コマンドライン引数）

  例えば、次のようなコマンドを呼び出したときの ``out`` は：

  .. code-block:: cmake

    separate_arguments (out UNIX_COMMAND PROGRAM "cc -c main.c")

  * ``<args>`` の先頭の要素: ``/path/to/cc``
  * ``<args>`` の二番目以降の要素: ``" -c main.c"``

``SEPARATE_ARGS``
  ``PROGRAM`` オプションで、このサブオプションを指定すると、``<args>`` をさらに分解して ``<variable>`` に格納する。

  例えば、次のようなコマンドを呼び出したとき：

  .. code-block:: cmake

    separate_arguments (out UNIX_COMMAND PROGRAM SEPARATE_ARGS "cc -c main.c")

   ``out`` の中身は： ``/path/to/cc;-c;main.c``

.. _`Parsing C Command-Line Arguments`: https://learn.microsoft.com/en-us/cpp/c-language/parsing-c-command-line-arguments

.. code-block:: cmake

  separate_arguments(<var>)

``<var>`` に格納された値を「:ref:`セミコロンで区切られたリスト <CMake Language Lists>`」に変換します。
空白はすべてセミコロン（``;``）に置き換える。
このコマンドは、変数からコマンドライン引数のリストを作成する際に便利です。
