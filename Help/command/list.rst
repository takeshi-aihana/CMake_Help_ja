list
----

:ref:`セミコロンで区切られたリスト <CMake Language Lists>` を操作する。

概要
^^^^

.. parsed-literal::

  `リストを読み取る`_
    list(`LENGTH`_ <list> <out-var>)
    list(`GET`_ <list> <element index> [<index> ...] <out-var>)
    list(`JOIN`_ <list> <glue> <out-var>)
    list(`SUBLIST`_ <list> <begin> <length> <out-var>)

  `リストを検索する`_
    list(`FIND`_ <list> <value> <out-var>)

  `リストを変更する`_
    list(`APPEND`_ <list> [<element>...])
    list(`FILTER`_ <list> {INCLUDE | EXCLUDE} REGEX <regex>)
    list(`INSERT`_ <list> <index> [<element>...])
    list(`POP_BACK`_ <list> [<out-var>...])
    list(`POP_FRONT`_ <list> [<out-var>...])
    list(`PREPEND`_ <list> [<element>...])
    list(`REMOVE_ITEM`_ <list> <value>...)
    list(`REMOVE_AT`_ <list> <index>...)
    list(`REMOVE_DUPLICATES`_ <list>)
    list(`TRANSFORM`_ <list> <ACTION> [...])

  `リストの要素の順番`_
    list(`REVERSE`_ <list>)
    list(`SORT`_ <list> [...])

はじめに
^^^^^^^^

このコマンドの :cref:`APPEND`、:cref:`INSERT`、:cref:`FILTER`、:cref:`PREPEND`、:cref:`POP_BACK`、:cref:`POP_FRONT`、:cref:`REMOVE_AT`、:cref:`REMOVE_ITEM`、:cref:`REMOVE_DUPLICATES`、:cref:`REVERSE`、:cref:`SORT` といったサブコマンドは、現在の CMake 変数のスコープの中で「リスト」に新しい要素を作成できます。
:command:`set` コマンドと同様に、``list`` コマンドはリスト自体が実際には親スコープで定義されたものであっても、現在のスコープに対する値を作成します。
リストを操作したときの結果を、上位のスコープに伝搬させる場合は、``PARENT_SCOPE`` や ``CACHE INTERNAL`` を指定した :command:`set` コマンド（あるいはその他の手段）を使います。

.. note::

  CMake の「リスト」はセミコロン（``;``）で区切った文字列を要素とするグループです。
  リストを生成するには :command:`set` コマンドを使用します。
  例えば ``set(var a b c d e)`` というコマンドは ``a;b;c;d;e`` というリストを作成し、``set(var "a b c d e")`` というコマンドは文字列または一個の要素からなるリストを作成します。
  マクロで使う引数（``ARGN`` や ``ARGC`` や ``ARGV`` や ``ARGV0`` の類）は変数ではないので、``list`` コマンドでは使用できない点に注意して下さい。

  それぞれの要素に ``[`` と ``]`` 文字を含めることはできません。さらに要素の末尾をバックスラッシュ (``\``) にすることはできません。
  詳細は :ref:`セミコロンで区切られたリスト <CMake Language Lists>` を参照して下さい。

.. note::

  リストで「インデックス」を指定する際、 ``<element index>`` が 0 以上の場合、リストの先頭からインデックスが付与され、0 がリストで先頭の要素を表します。
  ``<element index>`` が -1 以下の場合、リストの末尾からインデックスが付与され、-1 がリストで最後の要素を表します。
  後者の負のインデックスでカウントする際は注意が必要です： インデックスは 0 から始まりません。
  -0 の要素は、リストの先頭の要素でインデックスが 0 の要素と等価です。

リストを読み取る
^^^^^^^^^^^^^^^^

.. signature::
  list(LENGTH <list> <output variable>)

  ``<list>`` のサイズを返す。

.. signature::
  list(GET <list> <element index> [<element index> ...] <output variable>)

  ``<list>`` から ``<element index> ...`` のインデックスを持つ要素 ... をリストで返す。

.. signature:: list(JOIN <list> <glue> <output variable>)

  .. versionadded:: 3.12

  ``<glue>`` 文字列を使って、``<list>`` にある全ての要素を連結し、その文字列を返す。
  なお、リストではない複数の文字列を連結する場合は :command:`string(JOIN)` コマンドを使うこと。

.. signature::
  list(SUBLIST <list> <begin> <length> <output variable>)

  .. versionadded:: 3.12

  ``<list>`` のサブリストを返す。
  ``<length>`` が 0 の場合は空のリストを返す。
  ``<length>`` が -1、またはリストのサイズが ``<begin>+<length>`` よりも小さい場合、``<begin>`` から始まる残りの要素をリストで返す。

リストを検索する
^^^^^^^^^^^^^^^^

.. signature::
  list(FIND <list> <value> <output variable>)

  ``<list>`` から ``<value>`` と同じ要素のインデックスを返す。または見つからなければ ``-1`` を返す。

リストを変更する
^^^^^^^^^^^^^^^^

.. signature::
  list(APPEND <list> [<element> ...])

  ``<list>`` の最後に ``<element> ...`` を追加する。
  現在のスコープに ``<list>`` という名前の変数が存在していない場合、空の ``<list>`` を作成して、そのリストに追加する。

.. signature::
  list(FILTER <list> <INCLUDE|EXCLUDE> REGEX <regular_expression>)

.. versionadded:: 3.6

パタンにマッチする要素を ``<list>`` に追加したり ``<list>`` から削除する。
``REGEX`` モードの場合は、指定した正規表現 ``<regular_expression>`` にマッチする要素が対象となる。

正規表現について詳細は :ref:`string(REGEX) <Regex Specification>` を参照のこと。

.. signature::
  list(INSERT <list> <element_index> <element> [<element> ...])

  ``<list>`` の ``<element_index>`` の位置に ``<element> ...`` を挿入する。
  指定した ``<element_index>`` が ``<list>`` 外のインデックスの場合はエラーになる。
  ここで有効なインデックスは 0 から `N`（ `N` は ``<list>`` のサイズ）。
  空のリストのサイズは 0 である。
  現在のスコープに ``<list>`` という名前の変数が存在していない場合、空の ``<list>`` を作成して、その空のリストに挿入する。

.. signature::
  list(POP_BACK <list> [<out-var>...])

  .. versionadded:: 3.15

  ``<out-var> ...`` を指定しない場合は、要素を１つだけ削除する。
  `N` 個の ``<out-var>`` が指定されている場合、最後に ``N`` 個目の要素の値を変数にセットしてから、その要素を ``<list>`` から削除する。

.. signature::
  list(POP_FRONT <list> [<out-var>...])

  .. versionadded:: 3.15

  ``<out-var> ...`` を指定しない場合は、要素を１つだけ削除する。
  `N` 個の ``<out-var>`` が指定されている場合、最初に ``N`` 個目の要素の値を変数にセットしてから、その要素を ``<list>`` から削除する。

.. signature::
  list(PREPEND <list> [<element> ...])

  .. versionadded:: 3.15

  ``<list>`` の 0番目の位置に ``<element> ...`` を挿入する。
  現在のスコープに ``<list>`` という名前の変数が存在していない場合、空の ``<list>`` を作成して、その空のリストの先頭に挿入する。

.. signature::
  list(REMOVE_ITEM <list> <value> [<value> ...])

  ``<list>`` から ``<value> ...`` に該当する要素を全て削除する。

.. signature::
  list(REMOVE_AT <list> <index> [<index> ...])

  ``<list>`` から ``<index> ...`` の位置にある要素を全て削除する。

.. signature::
  list(REMOVE_DUPLICATES <list>)

  ``<list>`` で重複している要素を削除する。
  要素の相対的な順番は保持するが、もし重複した要素が見つかったら、最初の要素だけ残し、それ以外を全て削除する。

.. signature::
  list(TRANSFORM <list> <ACTION> [<SELECTOR>]
       [OUTPUT_VARIABLE <output variable>])

  .. versionadded:: 3.12

  ``<list>`` の中の全ての要素、または ``<SELECTOR>`` で選択された要素に ``<ACTION>`` を適用し、その結果を出力するか、または ``<output variable>`` に保存する。

  .. note::

    この ``TRANSFORM`` サブコマンドは要素の総数は変更しません。
    ``<SELECTOR>`` を指定した場合、一部の要素だけ変更し、残りの要素は変換前のままです。

  ``<ACTION>`` には、リストの中にある要素に適用する「アクション」を指定する。
  このアクションは  :command:`string` コマンドのサブコマンドと同じである。
  すなわち ``<ACTION>`` には以下のいずれかを指定すること：

    :command:`APPEND <string(APPEND)>`, :command:`PREPEND <string(PREPEND)>`
      指定した値を要素の最後に追加する、または要素の先頭に追加する。

      .. signature::
        list(TRANSFORM <list> (APPEND|PREPEND) <value> ...)
        :target: TRANSFORM_APPEND

    :command:`TOLOWER <string(TOLOWER)>`, :command:`TOUPPER <string(TOUPPER)>`
      要素の小文字を大文字に、大文字を小文字にそれぞれ変換する。

      .. signature::
        list(TRANSFORM <list> (TOLOWER|TOUPPER) ...)
        :target: TRANSFORM_TOLOWER

    :command:`STRIP <string(STRIP)>`
      要素の先頭と末尾にある空白を削除する。

      .. signature::
        list(TRANSFORM <list> STRIP ...)
        :target: TRANSFORM_STRIP

    :command:`GENEX_STRIP <string(GENEX_STRIP)>`
      要素から「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を削除する。

      .. signature::
        list(TRANSFORM <list> GENEX_STRIP ...)
        :target: TRANSFORM_GENEX_STRIP

    :command:`REPLACE <string(REGEX REPLACE)>`:
      要素で正規表現にマッチする部分を置換する（:command:`string(REGEX REPLACE)` コマンドと同じ機能）。

      .. signature::
        list(TRANSFORM <list> REPLACE <regular_expression>
                                      <replace_expression> ...)
        :target: TRANSFORM_REPLACE

  ``<SELECTOR>`` で、リストの中にあるどの要素にアクションを適用するかを決定する。
  一度に指定できる ``<SELECTOR>`` は１個だけで、以下のいずれかを指定する：

    ``AT``
      インデックスで要素を選択するため、インデックスを格納したリストを渡す。

      .. code-block:: cmake

        list(TRANSFORM <list> <ACTION> AT <index> [<index> ...] ...)

    ``FOR``
      範囲で要素を選択するため、先頭のインデックスと最後のインデックスを渡す（オプションで間隔を指定できる）。

      .. code-block:: cmake

        list(TRANSFORM <list> <ACTION> FOR <start> <stop> [<step>] ...)

    ``REGEX``
      正規表現で要素を選択するため、:ref:`string(REGEX) <Regex Specification>` を指定する。
      この正規表現にマッチする要素のみアクションが適用される。

      .. code-block:: cmake

        list(TRANSFORM <list> <ACTION> REGEX <regular_expression> ...)


リストの要素の順番
^^^^^^^^^^^^^^^^^^

.. signature::
  list(REVERSE <list>)

  ``<list>`` の要素の順番を（その場で）反転する。

.. signature::
  list(SORT <list> [COMPARE <compare>] [CASE <case>] [ORDER <order>])

  ``<list>`` の要素を（その場で）アルファベット順に並べ替える。

  .. versionadded:: 3.13
    ``COMPARE`` と ``CASE`` と ``ORDER`` のオプションが追加された。

  .. versionadded:: 3.18
    ``COMPARE NATURAL``  のオプションが追加された。

  ``COMPARE`` オプションの ``<compare>`` で、並び替える際の比較方法を指定する。
  ``<compare>`` には、以下のいずれかを指定する：

    ``STRING``
      要素の値を文字列として比較して並び替える（これが ``COMPARE`` オプションを指定しなかった場合のデフォルト）。

    ``FILE_BASENAME``
    要素の値をパスのベース名として比較して並び替える。

    ``NATURAL``
      要素の値を数値として比較して並び替える（``strverscmp(3)`` のマニュアルを参照）。
      例： 次のリスト `10.0 1.1 2.1 8.0 2.0 3.1` は、``NATURAL`` 方式だと `1.1 2.0 2.1 3.1 8.0 10.0` に並び替えられ、``STRING`` 方式だと `1.1 10.0 2.0 2.1 3.1 8.0` に並び替えられる。

  ``CASE`` オプションの ``<case>`` で大文字と小文字を区別するかどうかを指定する。
  ``<case>`` には、以下のいずれかを指定する：

    ``SENSITIVE``
      要素は大文字／小文字を区別する（これが ``CASE`` オプションを指定しなかった場合のデフォルト）。

    ``INSENSITIVE``
      要素は大文字／小文字を区別しない。
      要素の違いが大文字か小文字であるかだけの場合は順序を変えない。

  並び替える順番は ``ORDER`` オプションの ``<order>`` で指定できる。
  ``<order>`` には、以下のいずれかを指定する：

    ``ASCENDING``
      要素を昇順に並び替える（これが ``ORDER`` オプションを指定しなかった場合のデフォルト）。

    ``DESCENDING``
      要素を降順に並び替える。
