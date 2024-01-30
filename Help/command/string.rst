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

  `Comparison`_
    string(`COMPARE`_ <op> <string1> <string2> <out-var>)

  `Hashing`_
    string(`\<HASH\>`_ <out-var> <input>)

  `Generation`_
    string(`ASCII`_ <number>... <out-var>)
    string(`HEX`_ <string> <out-var>)
    string(`CONFIGURE`_ <string> <out-var> [...])
    string(`MAKE_C_IDENTIFIER`_ <string> <out-var>)
    string(`RANDOM`_ [<option>...] <out-var>)
    string(`TIMESTAMP`_ <out-var> [<format string>] [UTC])
    string(`UUID`_ <out-var> ...)

  `JSON`_
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

以下の文字は「正規表現（*Regular Expression*）」のパタンにおいて特別な意味があります：

``^``
  ``<input>`` の先頭にマッチする。
``$``
  ``<input>`` の末尾にマッチする。
``.``
  ``<input>`` にある一個の文字にマッチする。
``\<char>``
  ``<char>`` という一個のリテラルの文字にマッチする。
  これを使用して、特殊な文字にマッチする（  例えば： ``\.`` は一個のリテラルの文字にマッチし、``\\`` は一個のバックスラッシュ（``\``）にマッチする）。
  一般的に特殊文字以外のエスケープは不要である（ただし利用はできる： ``\a`` は ``a`` にマッチする）。
``[ ]``
  カッコの中にある任意の文字にマッチする。
``[^ ]``
  カッコの中にない任意の文字にマッチする。
``-``
  カッコの中では、この両端にある文字の範囲を表す（例えば：. ``[a-f]`` は ``[abcdef]``）。
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

  Append all the ``<input>`` arguments to the string.

.. signature::
  string(PREPEND <string_variable> [<input>...])

  .. versionadded:: 3.10

  Prepend all the ``<input>`` arguments to the string.

.. signature::
  string(CONCAT <output_variable> [<input>...])

  Concatenate all the ``<input>`` arguments together and store
  the result in the named ``<output_variable>``.

.. signature::
  string(JOIN <glue> <output_variable> [<input>...])

  .. versionadded:: 3.12

  Join all the ``<input>`` arguments together using the ``<glue>``
  string and store the result in the named ``<output_variable>``.

  To join a list's elements, prefer to use the ``JOIN`` operator
  from the :command:`list` command.  This allows for the elements to have
  special characters like ``;`` in them.

.. signature::
  string(TOLOWER <string> <output_variable>)

  Convert ``<string>`` to lower characters.

.. signature::
  string(TOUPPER <string> <output_variable>)

  Convert ``<string>`` to upper characters.

.. signature::
  string(LENGTH <string> <output_variable>)

  Store in an ``<output_variable>`` a given string's length in bytes.
  Note that this means if ``<string>`` contains multi-byte characters,
  the result stored in ``<output_variable>`` will *not* be
  the number of characters.

.. signature::
  string(SUBSTRING <string> <begin> <length> <output_variable>)

  Store in an ``<output_variable>`` a substring of a given ``<string>``.  If
  ``<length>`` is ``-1`` the remainder of the string starting at ``<begin>``
  will be returned.

  .. versionchanged:: 3.2
    If ``<string>`` is shorter than ``<length>``
    then the end of the string is used instead.
    Previous versions of CMake reported an error in this case.

  Both ``<begin>`` and ``<length>`` are counted in bytes, so care must
  be exercised if ``<string>`` could contain multi-byte characters.

.. signature::
  string(STRIP <string> <output_variable>)

  Store in an ``<output_variable>`` a substring of a given ``<string>``
  with leading and trailing spaces removed.

.. signature::
  string(GENEX_STRIP <string> <output_variable>)

  .. versionadded:: 3.1

  Strip any :manual:`generator expressions <cmake-generator-expressions(7)>`
  from the input ``<string>`` and store the result
  in the ``<output_variable>``.

.. signature::
  string(REPEAT <string> <count> <output_variable>)

  .. versionadded:: 3.15

  Produce the output string as the input ``<string>``
  repeated ``<count>`` times.

Comparison
^^^^^^^^^^

.. _COMPARE:

.. signature::
  string(COMPARE LESS <string1> <string2> <output_variable>)
  string(COMPARE GREATER <string1> <string2> <output_variable>)
  string(COMPARE EQUAL <string1> <string2> <output_variable>)
  string(COMPARE NOTEQUAL <string1> <string2> <output_variable>)
  string(COMPARE LESS_EQUAL <string1> <string2> <output_variable>)
  string(COMPARE GREATER_EQUAL <string1> <string2> <output_variable>)

  Compare the strings and store true or false in the ``<output_variable>``.

  .. versionadded:: 3.7
    Added the ``LESS_EQUAL`` and ``GREATER_EQUAL`` options.

.. _`Supported Hash Algorithms`:

Hashing
^^^^^^^

.. signature::
  string(<HASH> <output_variable> <input>)
  :target: <HASH>

  Compute a cryptographic hash of the ``<input>`` string.
  The supported ``<HASH>`` algorithm names are:

  ``MD5``
    Message-Digest Algorithm 5, RFC 1321.
  ``SHA1``
    US Secure Hash Algorithm 1, RFC 3174.
  ``SHA224``
    US Secure Hash Algorithms, RFC 4634.
  ``SHA256``
    US Secure Hash Algorithms, RFC 4634.
  ``SHA384``
    US Secure Hash Algorithms, RFC 4634.
  ``SHA512``
    US Secure Hash Algorithms, RFC 4634.
  ``SHA3_224``
    Keccak SHA-3.
  ``SHA3_256``
    Keccak SHA-3.
  ``SHA3_384``
    Keccak SHA-3.
  ``SHA3_512``
    Keccak SHA-3.

  .. versionadded:: 3.8
    Added the ``SHA3_*`` hash algorithms.

Generation
^^^^^^^^^^

.. signature::
  string(ASCII <number> [<number> ...] <output_variable>)

  Convert all numbers into corresponding ASCII characters.

.. signature::
  string(HEX <string> <output_variable>)

  .. versionadded:: 3.18

  Convert each byte in the input ``<string>`` to its hexadecimal representation
  and store the concatenated hex digits in the ``<output_variable>``.
  Letters in the output (``a`` through ``f``) are in lowercase.

.. signature::
  string(CONFIGURE <string> <output_variable>
         [@ONLY] [ESCAPE_QUOTES])

  Transform a ``<string>`` like :command:`configure_file` transforms a file.

.. signature::
  string(MAKE_C_IDENTIFIER <string> <output_variable>)

  Convert each non-alphanumeric character in the input ``<string>`` to an
  underscore and store the result in the ``<output_variable>``.  If the first
  character of the ``<string>`` is a digit, an underscore will also be
  prepended to the result.

.. signature::
  string(RANDOM [LENGTH <length>] [ALPHABET <alphabet>]
         [RANDOM_SEED <seed>] <output_variable>)

  Return a random string of given ``<length>`` consisting of
  characters from the given ``<alphabet>``.  Default length is 5 characters
  and default alphabet is all numbers and upper and lower case letters.
  If an integer ``RANDOM_SEED`` is given, its value will be used to seed the
  random number generator.

.. signature::
  string(TIMESTAMP <output_variable> [<format_string>] [UTC])

  Write a string representation of the current date
  and/or time to the ``<output_variable>``.

  If the command is unable to obtain a timestamp, the ``<output_variable>``
  will be set to the empty string ``""``.

  The optional ``UTC`` flag requests the current date/time representation to
  be in Coordinated Universal Time (UTC) rather than local time.

  The optional ``<format_string>`` may contain the following format
  specifiers:

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

    The offset of the time zone from UTC, in hours and minutes,
    with format ``+hhmm`` or ``-hhmm``.

  ``%Z``
    .. versionadded:: 3.26

    The time zone name.

  Unknown format specifiers will be ignored and copied to the output
  as-is.

  If no explicit ``<format_string>`` is given, it will default to:

  ::

    %Y-%m-%dT%H:%M:%S    for local time.
    %Y-%m-%dT%H:%M:%SZ   for UTC.

  .. versionadded:: 3.8
    If the ``SOURCE_DATE_EPOCH`` environment variable is set,
    its value will be used instead of the current time.
    See https://reproducible-builds.org/specs/source-date-epoch/ for details.

.. signature::
  string(UUID <output_variable> NAMESPACE <namespace> NAME <name>
         TYPE <MD5|SHA1> [UPPER])

  .. versionadded:: 3.1

  Create a universally unique identifier (aka GUID) as per RFC4122
  based on the hash of the combined values of ``<namespace>``
  (which itself has to be a valid UUID) and ``<name>``.
  The hash algorithm can be either ``MD5`` (Version 3 UUID) or
  ``SHA1`` (Version 5 UUID).
  A UUID has the format ``xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx``
  where each ``x`` represents a lower case hexadecimal character.
  Where required, an uppercase representation can be requested
  with the optional ``UPPER`` flag.

.. _JSON:

JSON
^^^^

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
