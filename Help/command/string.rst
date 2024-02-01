string
------

CMake の中で文字列を操作する。

概要
^^^^

.. parsed-literal::

  `文字列を検索し置換する`_
    string(`FIND`_ <string> <substring> <out-var> [...])
    string(`REPLACE`_ <match-string> <replace-string> <out-var> <input>...)
    string(`REGEX MATCH`_ <match-regex> <out-var> <input>...)
    string(`REGEX MATCHALL`_ <match-regex> <out-var> <input>...)
    string(`REGEX REPLACE`_ <match-regex> <replace-expr> <out-var> <input>...)

  `文字列を加工する`_
    string(`APPEND`_ <string-var> [<input>...])
    string(`PREPEND`_ <string-var> [<input>...])
    string(`CONCAT`_ <out-var> [<input>...])
    string(`JOIN`_ <glue> <out-var> [<input>...])
    string(`TOLOWER`_ <string> <out-var>)
    string(`TOUPPER`_ <string> <out-var>)
    string(`LENGTH`_ <string> <out-var>)
    string(`SUBSTRING`_ <string> <begin> <length> <out-var>)
    string(`STRIP`_ <string> <out-var>)
    string(`GENEX_STRIP`_ <string> <out-var>)
    string(`REPEAT`_ <string> <count> <out-var>)

  `文字列を比較する`_
    string(`COMPARE`_ <op> <string1> <string2> <out-var>)

  `文字列のハッシュ値を計算する`_
    string(`\<HASH\>`_ <out-var> <input>)

  `文字を変換する`_
    string(`ASCII`_ <number>... <out-var>)
    string(`HEX`_ <string> <out-var>)
    string(`CONFIGURE`_ <string> <out-var> [...])
    string(`MAKE_C_IDENTIFIER`_ <string> <out-var>)
    string(`RANDOM`_ [<option>...] <out-var>)
    string(`TIMESTAMP`_ <out-var> [<format string>] [UTC])
    string(`UUID`_ <out-var> ...)

  `文字列を JSON 化する`_
    string(JSON <out-var> [ERROR_VARIABLE <error-var>]
           {`GET <JSON GET_>`_ | `TYPE <JSON TYPE_>`_ | `LENGTH <JSON LENGTH_>`_ | `REMOVE <JSON REMOVE_>`_}
           <json-string> <member|index> [<member|index> ...])
    string(JSON <out-var> [ERROR_VARIABLE <error-var>]
           `MEMBER <JSON MEMBER_>`_ <json-string>
           [<member|index> ...] <index>)
    string(JSON <out-var> [ERROR_VARIABLE <error-var>]
           `SET <JSON SET_>`_ <json-string>
           <member|index> [<member|index> ...] <value>)
    string(JSON <out-var> [ERROR_VARIABLE <error-var>]
           `EQUAL <JSON EQUAL_>`_ <json-string1> <json-string2>)

検索と置換
^^^^^^^^^^

文字列を検索し置換する
""""""""""""""""""""""

.. signature::
  string(FIND <string> <substring> <output_variable> [REVERSE])

  ``<string>`` の中で ``<substring>`` が **最初に** 出現する位置を返します。
  一方で ``REVERSE`` オプションを指定すると、``<string>`` の中で ``<substring>`` が **最後に** 出現する位置を返します。
  ``<substring>`` が見つからなかったら、-1 を返します。

  この ``string(FIND)`` サブコマンドでは ASCII 文字のみを扱います。
  したがって ``<output_variable>`` に返された ``<substring>`` の位置（``<string>`` の中でのインデックス）はバイト単位でカウントします。
  そのため ``<string>`` や ``<substring>`` がマルチバイト文字を含む文字列の場合は想定しない結果になる場合があります。

.. signature::
  string(REPLACE <match_string>
         <replace_string> <output_variable>
         <input> [<input>...])

  ``<input>`` の中にあるすべての ``<match_string>`` を ``<replace_string>`` で置き換えて、その結果を ``<output_variable>`` に格納します。

正規表現を使った文字列の検索と置換
""""""""""""""""""""""""""""""""""

.. signature::
  string(REGEX MATCH <regular_expression>
         <output_variable> <input> [<input>...])

  ``<regular_expression>`` にマッチした結果を ``<output_variable>`` に格納します。
  引数の ``<input> ...`` は検索する前にすべて連結します。
  正規表現のパタンについては、このセクションにある「:ref:`Regex Specification`」を参照して下さい。

.. signature::
  string(REGEX MATCHALL <regular_expression>
         <output_variable> <input> [<input>...])

  ``<regular_expression>`` にマッチした全ての結果を「:ref:`セミコロンで区切られたリスト <CMake Language Lists>` 」にして ``<output_variable>`` に格納します。
  引数の ``<input> ...`` は検索する前にすべて連結します。

.. signature::
  string(REGEX REPLACE <regular_expression>
         <replacement_expression> <output_variable>
         <input> [<input>...])

  ``<regular_expression>`` にマッチした全ての結果を ``<replacement_expression>`` で置き換えます。
  引数の ``<input> ...`` は検索する前にすべて連結します。

  この ``<replacement_expression>`` は、``\1`` や ``\2``, ..., ``\9`` とカッコ（``()``）を使ってマッチした部分文字列を参照できます。
  一個のバックスラッシュ（``\``）にマッチさせたい場合は、二個のバックスラッシュ（``\\1``）が必要である点に留意して下さい。

.. _`Regex Specification`:

正規表現の仕様
""""""""""""""

ここにある文字は「正規表現（*Regular Expression*）」のパタンにおいて特別な意味を持ちます：

``^``
  ``<input>`` の先頭にマッチする。
``$``
  ``<input>`` の末尾にマッチする。
``.``
  ``<input>`` にある一個の文字にマッチする。
``\<char>``
  ``<char>`` という一個のリテラルの文字にマッチする。
  これを利用して、特殊な文字にマッチすることが可能である（例えば： ``\.`` は一個のリテラルの文字にマッチし、``\\`` は一個のバックスラッシュ（``\``）にマッチする）。
  一般的に特殊文字以外のエスケープは不要である（ただし利用は可能： 例えば ``\a`` は ``a`` にマッチする）。
``[ ]``
  カッコの中にある任意の文字にマッチする。
``[^ ]``
  カッコの中にない任意の文字にマッチする。
``-``
  カッコの中では、この両端にある文字でパタンの範囲を表す（例えば：. ``[a-f]`` は ``[abcdef]``）。
  リテラルの ``-`` にマッチさせるには、カッコを使用して、それを最初または最後に置く（例えば： ``[+*/-]`` は基本演算子のいずれかにマッチする）。
``*``
  これより前にある正規表現パタンに０回以上マッチする。
``+``
  これより前にある正規表現パタンに１回以上マッチする。
``?``
  これより前にある正規表現パタンに０回または１回だけマッチする。
``|``
  これのどちらか側にあるいずれかの正規表現のパタンにマッチする。
``()``
  正規表現パタンにマッチした部分文字列を保存する（保存したものは ``REGEX REPLACE`` 操作で参照できる）。

  .. versionadded:: 3.9
    正規表現を利用する全てのコマンド（:command:`if(MATCHES)` など）が、正規表現パタンにマッチした部分文字列を保存して、CMake 変数の :variable:`CMAKE_MATCH_<n>` （``<n>`` は 0..9） で参照できるようになった。

``*`` と ``+`` と ``?`` による検索は、文字列の連結よりも優先順位が高いです。
``|`` による検索は、文字列の連結よりも優先順位が低いです。

この仕様を使った例： ``^ab+d$`` という正規表現パタンは ``abbd`` にマッチしますが、``ababd`` にはマッチしません。``^(ab|cd)$`` という正規表現パタンは ``ab`` にマッチしますが、``abd`` にはマッチしません。

``\t`` や ``\r`` や ``\n`` や ``\\`` といった制御文字（エスケープ・シーケンス）を使用すると、順にタブ文字、復帰（キャリッジ・リターン）文字、改行（リターン）文字、バックスラッシュのリテラルをそれぞれ表現するパタンを構築できます。
例えば：

* 引用符で囲んだ ``"[ \t\r\n]"`` は一個の空白文字にマッチする正規表現パタンである。
* 引用符で囲んだ ``"[/\\]"`` は一個のスラッシュ（``/``）またはバックスラッシュ（``\``）にマッチする正規表現パタンである。
* 引用符で囲んだ ``"[A-Za-z0-9_]"`` はＣロケールで一個の単語にマッチする正規表現パタンである。
* 引用符で囲んだ ``"\\(\\a\\+b\\)"`` は文字列の ``(a+b)`` と完全にマッチする正規表現パタンである。
  この中にある ``\\`` はただのスペース（``\``）と認識されるので、このパタンは正確には ``"\(\a\+\b\)"`` である。
  これは、バックスラッシュをエスケープするかわりに :ref:`bracket argument` を使って ``"[[\(\a\+\b\)]]"`` で表現できる。

文字列を加工する
^^^^^^^^^^^^^^^^

.. signature::
  string(APPEND <string_variable> [<input>...])

  .. versionadded:: 3.4

  ``<string_variable>`` に格納された文字列の最後に、全ての ``<input>...`` を追加します。

.. signature::
  string(PREPEND <string_variable> [<input>...])

  .. versionadded:: 3.10

  ``<string_variable>`` に格納された文字列の先頭に、全ての ``<input>...`` を追加します。

.. signature::
  string(CONCAT <output_variable> [<input>...])

  全ての ``<input>...`` を連結して、その結果を ``<output_variable>`` に格納する。

.. signature::
  string(JOIN <glue> <output_variable> [<input>...])

  .. versionadded:: 3.12

  ``<glue>`` の文字列を使って、全ての ``<input>...`` を連結し、その結果を ``<output_variable>`` に格納する。

  :ref:`リスト <CMake Language Lists>` の要素を連結する場合は、:command:`list(JOIN)` コマンドを使用すること推奨します。
  これにより、要素に ``;`` のような特殊文字を含めることができます。

.. signature::
  string(TOLOWER <string> <output_variable>)

  ``<string>`` を小文字に変換します。

.. signature::
  string(TOUPPER <string> <output_variable>)

  ``<string>`` を大文字に変換します。

.. signature::
  string(LENGTH <string> <output_variable>)

  ``<string>`` の長さをバイト単位でカウントして ``<output_variable>`` に格納します。
  もし ``<string>`` にマルチバイトの文字が含まれている場合、``<output_variable>`` に格納された結果は正しい文字数ではないので注意して下さい。

.. signature::
  string(SUBSTRING <string> <begin> <length> <output_variable>)

  ``<string>`` の部分文字列を ``<output_variable>`` に格納します。
  ``<length>`` が ``-1`` 場合は、``<begin>`` で始まる ``<string>`` の残りの部分文字列を返します。

  .. versionchanged:: 3.2
    ``<string>`` の長さが ``<length>`` より短い場合は、``<string>`` の末尾の部分文字列を返すようになった。
    CMake の以前のバージョンではエラーを報告していた。

  ``<begin>`` と ``<length>`` の両方はどちらもバイト単位でカウントするので、``<string>`` にマルチバイトの文字が含まれている場合は注意が必要です。

.. signature::
  string(STRIP <string> <output_variable>)

  ``<string>`` の先頭と末尾の空白文字を取り除いた部分文字列を ``<output_variable>`` に格納します。

.. signature::
  string(GENEX_STRIP <string> <output_variable>)

  .. versionadded:: 3.1

  ``<string>`` から「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を取り除き、その結果を ``<output_variable>`` に格納します。

.. signature::
  string(REPEAT <string> <count> <output_variable>)

  .. versionadded:: 3.15

  ``<string>`` を ``<count>`` 回繰り返した文字列を ``<output_variable>`` に格納します。

文字列を比較する
^^^^^^^^^^^^^^^^

.. _COMPARE:

.. signature::
  string(COMPARE LESS <string1> <string2> <output_variable>)
  string(COMPARE GREATER <string1> <string2> <output_variable>)
  string(COMPARE EQUAL <string1> <string2> <output_variable>)
  string(COMPARE NOTEQUAL <string1> <string2> <output_variable>)
  string(COMPARE LESS_EQUAL <string1> <string2> <output_variable>)
  string(COMPARE GREATER_EQUAL <string1> <string2> <output_variable>)

  ``<string1>`` と ``<string2>`` を比較して、オプションとして指定した演算子に応じた結果（true または false）を ``<output_variable>`` に格納します。

  .. versionadded:: 3.7
    ``LESS_EQUAL`` と ``GREATER_EQUAL`` のオプションを追加した。

.. _`Supported Hash Algorithms`:

文字列のハッシュ値を計算する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. signature::
  string(<HASH> <output_variable> <input>)
  :target: <HASH>

  ``<input>`` の文字列に対する暗号化ハッシュ値を計算します。
  サポートしている ``<HASH>`` アルゴリズムは次のとおりです：

  ``MD5``
    Message-Digest アルゴリズム 5（RFC 1321）
  ``SHA1``
    US Secure Hash アルゴリズム 1（RFC 3174）
  ``SHA224``
    US Secure Hash アルゴリズム（RFC 4634）
  ``SHA256``
    US Secure Hash アルゴリズム（RFC 4634）
  ``SHA384``
    US Secure Hash アルゴリズム（RFC 4634）
  ``SHA512``
    US Secure Hash アルゴリズム（RFC 4634）
  ``SHA3_224``
    Keccak SHA-3
  ``SHA3_256``
    Keccak SHA-3
  ``SHA3_384``
    Keccak SHA-3
  ``SHA3_512``
    Keccak SHA-3

  .. versionadded:: 3.8
    ``SHA3_*`` 系のハッシュアルゴリズムを追加した。

文字を変換する
^^^^^^^^^^^^^^

.. signature::
  string(ASCII <number> [<number> ...] <output_variable>)

  全ての ``<number> ...`` を対応する ASCII 文字に変換します。

.. signature::
  string(HEX <string> <output_variable>)

  .. versionadded:: 3.18

  ``<string>`` にある各バイトを16進数表記に変換し、連結した16新表記の文字列を ``<output_variable>`` に格納します。
  16進数表記の文字（``a`` から ``f``） は小文字です。

.. signature::
  string(CONFIGURE <string> <output_variable>
         [@ONLY] [ESCAPE_QUOTES])

  :command:`configure_file`  コマンドの変換のように ``<string>`` を変換します。

.. signature::
  string(MAKE_C_IDENTIFIER <string> <output_variable>)

  ``<string>`` の中にある英数字以外の文字をアンダースコアに変換し、その結果を ``<output_variable>`` に格納します。
  ``<string>`` の最初の文字が数字の場合、変換結果の先頭はアンダースコアになります。

.. signature::
  string(RANDOM [LENGTH <length>] [ALPHABET <alphabet>]
         [RANDOM_SEED <seed>] <output_variable>)

  ``<alphabet>`` の文字種で構成される長さが ``<length>`` のランダムな文字列を生成して返します。
  デフォルトの長さは5文字で、デフォルトの文字種は英数字（大文字と小文字）です。
  ``RANDOM_SEED`` オプションを指定すると、``<seed>`` を乱数ジェネレータのシードに使用します。

.. signature::
  string(TIMESTAMP <output_variable> [<format_string>] [UTC])

  Write a string representation of the current date and/or time to the ``<output_variable>``.

  If the command is unable to obtain a timestamp, the ``<output_variable>`` will be set to the empty string ``""``.

  The optional ``UTC`` flag requests the current date/time representation to be in Coordinated Universal Time (UTC) rather than local time.

  The optional ``<format_string>`` may contain the following format specifiers:

  ``%%``
    .. versionadded:: 3.8

    A literal percent sign (%).

  ``%d``
    The day of the current month (01-31).

  ``%H``
    The hour on a 24-hour clock (00-23).

  ``%I``
    The hour on a 12-hour clock (01-12).

  ``%j``
    The day of the current year (001-366).

  ``%m``
    The month of the current year (01-12).

  ``%b``
    .. versionadded:: 3.7

    Abbreviated month name (e.g. Oct).

  ``%B``
    .. versionadded:: 3.10

    Full month name (e.g. October).

  ``%M``
    The minute of the current hour (00-59).

  ``%s``
    .. versionadded:: 3.6

    Seconds since midnight (UTC) 1-Jan-1970 (UNIX time).

  ``%S``
    The second of the current minute.  60 represents a leap second. (00-60)

  ``%f``
    .. versionadded:: 3.23

    The microsecond of the current second (000000-999999).

  ``%U``
    The week number of the current year (00-53).

  ``%V``
    .. versionadded:: 3.22

    The ISO 8601 week number of the current year (01-53).

  ``%w``
    The day of the current week. 0 is Sunday. (0-6)

  ``%a``
    .. versionadded:: 3.7

    Abbreviated weekday name (e.g. Fri).

  ``%A``
    .. versionadded:: 3.10

    Full weekday name (e.g. Friday).

  ``%y``
    The last two digits of the current year (00-99).

  ``%Y``
    The current year.

  ``%z``
    .. versionadded:: 3.26

    The offset of the time zone from UTC, in hours and minutes, with format ``+hhmm`` or ``-hhmm``.

  ``%Z``
    .. versionadded:: 3.26

    The time zone name.

  Unknown format specifiers will be ignored and copied to the output as-is.

  If no explicit ``<format_string>`` is given, it will default to:

  ::

    %Y-%m-%dT%H:%M:%S    for local time.
    %Y-%m-%dT%H:%M:%SZ   for UTC.

  .. versionadded:: 3.8
    If the ``SOURCE_DATE_EPOCH`` environment variable is set, its value will be used instead of the current time.
    See https://reproducible-builds.org/specs/source-date-epoch/ for details.

.. signature::
  string(UUID <output_variable> NAMESPACE <namespace> NAME <name>
         TYPE <MD5|SHA1> [UPPER])

  .. versionadded:: 3.1

  Create a universally unique identifier (aka GUID) as per RFC4122 based on the hash of the combined values of ``<namespace>`` (which itself has to be a valid UUID) and ``<name>``.
  The hash algorithm can be either ``MD5`` (Version 3 UUID) or ``SHA1`` (Version 5 UUID).
  A UUID has the format ``xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`` where each ``x`` represents a lower case hexadecimal character.
  Where required, an uppercase representation can be requested with the optional ``UPPER`` flag.

.. _JSON:

文字列を JSON 化する
^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.19

Functionality for querying a JSON string.

.. note::
  In each of the following JSON-related subcommands, if the optional
  ``ERROR_VARIABLE`` argument is given, errors will be reported in
  ``<error-variable>`` and the ``<out-var>`` will be set to
  ``<member|index>-[<member|index>...]-NOTFOUND`` with the path elements
  up to the point where the error occurred, or just ``NOTFOUND`` if there
  is no relevant path.  If an error occurs but the ``ERROR_VARIABLE``
  option is not present, a fatal error message is generated.  If no error
  occurs, the ``<error-variable>`` will be set to ``NOTFOUND``.

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         GET <json-string> <member|index> [<member|index> ...])
  :target: JSON GET

  Get an element from ``<json-string>`` at the location given
  by the list of ``<member|index>`` arguments.
  Array and object elements will be returned as a JSON string.
  Boolean elements will be returned as ``ON`` or ``OFF``.
  Null elements will be returned as an empty string.
  Number and string types will be returned as strings.

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         TYPE <json-string> <member|index> [<member|index> ...])
  :target: JSON TYPE

  Get the type of an element in ``<json-string>`` at the location
  given by the list of ``<member|index>`` arguments. The ``<out-var>``
  will be set to one of ``NULL``, ``NUMBER``, ``STRING``, ``BOOLEAN``,
  ``ARRAY``, or ``OBJECT``.

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-var>]
         MEMBER <json-string>
         [<member|index> ...] <index>)
  :target: JSON MEMBER

  Get the name of the ``<index>``-th member in ``<json-string>``
  at the location given by the list of ``<member|index>`` arguments.
  Requires an element of object type.

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         LENGTH <json-string> [<member|index> ...])
  :target: JSON LENGTH

  Get the length of an element in ``<json-string>`` at the location
  given by the list of ``<member|index>`` arguments.
  Requires an element of array or object type.

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         REMOVE <json-string> <member|index> [<member|index> ...])
  :target: JSON REMOVE

  Remove an element from ``<json-string>`` at the location
  given by the list of ``<member|index>`` arguments. The JSON string
  without the removed element will be stored in ``<out-var>``.

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         SET <json-string> <member|index> [<member|index> ...] <value>)
  :target: JSON SET

  Set an element in ``<json-string>`` at the location
  given by the list of ``<member|index>`` arguments to ``<value>``.
  The contents of ``<value>`` should be valid JSON.
  If ``<json-string>`` is an array, ``<value>`` can be appended to the end of
  the array by using a number greater or equal to the array length as the
  ``<member|index>`` argument.

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-var>]
         EQUAL <json-string1> <json-string2>)
  :target: JSON EQUAL

  Compare the two JSON objects given by ``<json-string1>``
  and ``<json-string2>`` for equality.  The contents of ``<json-string1>``
  and ``<json-string2>`` should be valid JSON.  The ``<out-var>``
  will be set to a true value if the JSON objects are considered equal,
  or a false value otherwise.
