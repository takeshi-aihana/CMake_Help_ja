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
  マクロは変数ではないので、``list`` コマンドでは使用できない点に注意して下さい。

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

  Appends elements to the list. If no variable named ``<list>`` exists in the
  current scope its value is treated as empty and the elements are appended to
  that empty list.

.. signature::
  list(FILTER <list> <INCLUDE|EXCLUDE> REGEX <regular_expression>)

.. versionadded:: 3.6

Includes or removes items from the list that match the mode's pattern.
In ``REGEX`` mode, items will be matched against the given regular expression.

For more information on regular expressions look under
:ref:`string(REGEX) <Regex Specification>`.

.. signature::
  list(INSERT <list> <element_index> <element> [<element> ...])

  Inserts elements to the list to the specified index. It is an
  error to specify an out-of-range index. Valid indexes are 0 to `N`
  where `N` is the length of the list, inclusive. An empty list
  has length 0. If no variable named ``<list>`` exists in the
  current scope its value is treated as empty and the elements are
  inserted in that empty list.

.. signature::
  list(POP_BACK <list> [<out-var>...])

  .. versionadded:: 3.15

  If no variable name is given, removes exactly one element. Otherwise,
  with `N` variable names provided, assign the last `N` elements' values
  to the given variables and then remove the last `N` values from
  ``<list>``.

.. signature::
  list(POP_FRONT <list> [<out-var>...])

  .. versionadded:: 3.15

  If no variable name is given, removes exactly one element. Otherwise,
  with `N` variable names provided, assign the first `N` elements' values
  to the given variables and then remove the first `N` values from
  ``<list>``.

.. signature::
  list(PREPEND <list> [<element> ...])

  .. versionadded:: 3.15

  Insert elements to the 0th position in the list. If no variable named
  ``<list>`` exists in the current scope its value is treated as empty and
  the elements are prepended to that empty list.

.. signature::
  list(REMOVE_ITEM <list> <value> [<value> ...])

  Removes all instances of the given items from the list.

.. signature::
  list(REMOVE_AT <list> <index> [<index> ...])

  Removes items at given indices from the list.

.. signature::
  list(REMOVE_DUPLICATES <list>)

  Removes duplicated items in the list. The relative order of items
  is preserved, but if duplicates are encountered,
  only the first instance is preserved.

.. signature::
  list(TRANSFORM <list> <ACTION> [<SELECTOR>]
       [OUTPUT_VARIABLE <output variable>])

  .. versionadded:: 3.12

  Transforms the list by applying an ``<ACTION>`` to all or, by specifying a
  ``<SELECTOR>``, to the selected elements of the list, storing the result
  in-place or in the specified output variable.

  .. note::

    The ``TRANSFORM`` sub-command does not change the number of elements in the
    list. If a ``<SELECTOR>`` is specified, only some elements will be changed,
    the other ones will remain the same as before the transformation.

  ``<ACTION>`` specifies the action to apply to the elements of the list.
  The actions have exactly the same semantics as sub-commands of the
  :command:`string` command.  ``<ACTION>`` must be one of the following:

    :command:`APPEND <string(APPEND)>`, :command:`PREPEND <string(PREPEND)>`
      Append, prepend specified value to each element of the list.

      .. signature::
        list(TRANSFORM <list> (APPEND|PREPEND) <value> ...)
        :target: TRANSFORM_APPEND

    :command:`TOLOWER <string(TOLOWER)>`, :command:`TOUPPER <string(TOUPPER)>`
      Convert each element of the list to lower, upper characters.

      .. signature::
        list(TRANSFORM <list> (TOLOWER|TOUPPER) ...)
        :target: TRANSFORM_TOLOWER

    :command:`STRIP <string(STRIP)>`
      Remove leading and trailing spaces from each element of the list.

      .. signature::
        list(TRANSFORM <list> STRIP ...)
        :target: TRANSFORM_STRIP

    :command:`GENEX_STRIP <string(GENEX_STRIP)>`
      Strip any
      :manual:`generator expressions <cmake-generator-expressions(7)>`
      from each element of the list.

      .. signature::
        list(TRANSFORM <list> GENEX_STRIP ...)
        :target: TRANSFORM_GENEX_STRIP

    :command:`REPLACE <string(REGEX REPLACE)>`:
      Match the regular expression as many times as possible and substitute
      the replacement expression for the match for each element of the list
      (same semantic as :command:`string(REGEX REPLACE)`).

      .. signature::
        list(TRANSFORM <list> REPLACE <regular_expression>
                                      <replace_expression> ...)
        :target: TRANSFORM_REPLACE

  ``<SELECTOR>`` determines which elements of the list will be transformed.
  Only one type of selector can be specified at a time.
  When given, ``<SELECTOR>`` must be one of the following:

    ``AT``
      Specify a list of indexes.

      .. code-block:: cmake

        list(TRANSFORM <list> <ACTION> AT <index> [<index> ...] ...)

    ``FOR``
      Specify a range with, optionally,
      an increment used to iterate over the range.

      .. code-block:: cmake

        list(TRANSFORM <list> <ACTION> FOR <start> <stop> [<step>] ...)

    ``REGEX``
      Specify a regular expression.
      Only elements matching the regular expression will be transformed.

      .. code-block:: cmake

        list(TRANSFORM <list> <ACTION> REGEX <regular_expression> ...)


リストの要素の順番
^^^^^^^^^^^^^^^^^^

.. signature::
  list(REVERSE <list>)

  Reverses the contents of the list in-place.

.. signature::
  list(SORT <list> [COMPARE <compare>] [CASE <case>] [ORDER <order>])

  Sorts the list in-place alphabetically.

  .. versionadded:: 3.13
    Added the ``COMPARE``, ``CASE``, and ``ORDER`` options.

  .. versionadded:: 3.18
    Added the ``COMPARE NATURAL`` option.

  Use the ``COMPARE`` keyword to select the comparison method for sorting.
  The ``<compare>`` option should be one of:

    ``STRING``
      Sorts a list of strings alphabetically.
      This is the default behavior if the ``COMPARE`` option is not given.

    ``FILE_BASENAME``
      Sorts a list of pathnames of files by their basenames.

    ``NATURAL``
      Sorts a list of strings using natural order
      (see ``strverscmp(3)`` manual), i.e. such that contiguous digits
      are compared as whole numbers.
      For example: the following list `10.0 1.1 2.1 8.0 2.0 3.1`
      will be sorted as `1.1 2.0 2.1 3.1 8.0 10.0` if the ``NATURAL``
      comparison is selected where it will be sorted as
      `1.1 10.0 2.0 2.1 3.1 8.0` with the ``STRING`` comparison.

  Use the ``CASE`` keyword to select a case sensitive or case insensitive
  sort mode.  The ``<case>`` option should be one of:

    ``SENSITIVE``
      List items are sorted in a case-sensitive manner.
      This is the default behavior if the ``CASE`` option is not given.

    ``INSENSITIVE``
      List items are sorted case insensitively.  The order of
      items which differ only by upper/lowercase is not specified.

  To control the sort order, the ``ORDER`` keyword can be given.
  The ``<order>`` option should be one of:

    ``ASCENDING``
      Sorts the list in ascending order.
      This is the default behavior when the ``ORDER`` option is not given.

    ``DESCENDING``
      Sorts the list in descending order.
