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

  `JSON 形式を扱う`_
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

  引数の ``<input> ...`` に対して ``<regular_expression>`` にマッチした結果を ``<output_variable>`` に格納します。
  引数の ``<input> ...`` は検索する前にすべて連結します。
  正規表現のパタンについては、このセクションにある「:ref:`Regex Specification`」を参照して下さい。

.. signature::
  string(REGEX MATCHALL <regular_expression>
         <output_variable> <input> [<input>...])

  引数の ``<input> ...`` に対して ``<regular_expression>`` にマッチした全ての結果を「:ref:`セミコロンで区切られたリスト <CMake Language Lists>` 」にして ``<output_variable>`` に格納します。
  引数の ``<input> ...`` は検索する前にすべて連結します。

.. signature::
  string(REGEX REPLACE <regular_expression>
         <replacement_expression> <output_variable>
         <input> [<input>...])

  引数の ``<input> ...`` に対して ``<regular_expression>`` にマッチした全ての結果を ``<replacement_expression>`` で置き換えます。
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
    ``SHA3_*`` 系の Hash アルゴリズムを追加した。

文字を変換する
^^^^^^^^^^^^^^

.. signature::
  string(ASCII <number> [<number> ...] <output_variable>)

  全ての ``<number> ...`` の文字を対応する ASCII 文字に変換します。

.. signature::
  string(HEX <string> <output_variable>)

  .. versionadded:: 3.18

  ``<string>`` にある各バイトを16進数表記に変換し、連結した16新表記の文字列を ``<output_variable>`` に格納します。
  16進数表記の文字（``a`` から ``f``） は小文字になります。

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

  ``<alphabet>`` の文字種で構成され、長さが ``<length>`` のランダムな文字列を生成して返します。
  デフォルトの長さは5文字で、デフォルトの文字種は英数字（大文字と小文字の両方）です。
  ``RANDOM_SEED`` オプションを指定すると、``<seed>`` を乱数ジェネレータのシードに使用します。

.. signature::
  string(TIMESTAMP <output_variable> [<format_string>] [UTC])

  現在の日付および／または時刻の文字列表現を ``<output_variable>`` に格納します。

  このコマンドがタイムスタンプを取得できない場合、``<output_variable>`` には空の文字列（``""``）を格納します。

  ``UTC`` オプションを指定すると、現在の日付／時刻の表現が現在時刻ではなく、協定世界時（UTC）として要求します。

  ``<format_string>`` には、次に示す書式指定子を含めることができます：

  ``%%``
    .. versionadded:: 3.8

    リテラルとしてのパーセント記号（``%``）を表す。

  ``%d``
    月の初めからカウントした現在の日（``01``〜``31``）を表す。

  ``%H``
    24時間制で、現在の時（``00``〜``23``）を表す。

  ``%I``
    12時間制で、現在の時（``01``〜``12``）を表す。

  ``%j``
    年の初めからカウントした日（``001``〜``366``）を表す。

  ``%m``
    月（``01``〜``12``）を表す。

  ``%b``
    .. versionadded:: 3.7

    月の略称（例えば Oct）を表す。

  ``%B``
    .. versionadded:: 3.10

    月の完全な名前（例えば October）を表す。

  ``%M``
    現在の分（``01``〜``59``）を表す。

  ``%s``
    .. versionadded:: 3.6

    1970年1月1日の午前0時（UTC）からの秒数（UNIX 時間）を表す。

  ``%S``
    現在の秒（``01``〜``60``）を表す。``60`` は閏秒を表す。

  ``%f``
    .. versionadded:: 3.23

    現在のマイクロ秒（``000000``〜``999999``）を表す。

  ``%U``
    年の初めからカウントした週番号（``00``〜``53``）を表す。

  ``%V``
    .. versionadded:: 3.22

    ISO 8601 形式での年の始めからカウントした週番号（``01``〜``53``）を表す。

  ``%w``
    週の初めからカウントした日（``0``〜``6``）を表す。日曜日が ``0``。

  ``%a``
    .. versionadded:: 3.7

    曜日の略称（例えば Fri）を表す。

  ``%A``
    .. versionadded:: 3.10

    曜日の完全な名前（例えば Friday）を表す。

  ``%y``
    西暦の下2桁（``00``〜``99``）を表す。

  ``%Y``
    現在の年を表す。

  ``%z``
    .. versionadded:: 3.26

    UTC からのタイムゾーンのオフセット値（時と分ごと）を表す。形式は ``+hhmm`` または ``-hhmm``。

  ``%Z``
    .. versionadded:: 3.26

    タイムゾーンの名前。

  これ以外の不明な書式指定子は無視され、そのまま ``<output_variable>`` に格納します。

  明示的に ``<format_string>`` を指定しない場合、デフォルトの文字列表現は次のとおりです：

  ::

    %Y-%m-%dT%H:%M:%S    現在地の時間
    %Y-%m-%dT%H:%M:%SZ   UTC の時間

  .. versionadded:: 3.8
    環境変数 ``SOURCE_DATE_EPOCH`` が設定されている場合、現在の時刻の代わりにその値を使用する。
    詳細は https://reproducible-builds.org/specs/source-date-epoch/ を参照のこと。

.. signature::
  string(UUID <output_variable> NAMESPACE <namespace> NAME <name>
         TYPE <MD5|SHA1> [UPPER])

  .. versionadded:: 3.1

  UUID として有効な ``<namespace>`` と ``<name>`` の値を組み合わせて計算したハッシュ値に基づいて、RFC4122 に従い、汎用一意別子（別名は GUID）を生成します。
  Hash アルゴリズムは ``MD5`` (Version 3 UUID) または ``SHA1`` (Version 5 UUID) のいずれかを使用します。
  UUID の書式は ``xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`` （``x`` は 16進形式の小文字）です。
  ``UPPER`` オプションを指定すると16進形式の大文字を要求できます。

.. _JSON:

JSON 形式を扱う
^^^^^^^^^^^^^^^

.. versionadded:: 3.19

JSON 形式の文字列をクエリするサブコマンドが追加された。

.. note::
  ここにある JSON 関連のサブコマンドを呼び出す際に ``ERROR_VARIABLE`` オプションが指定されている場合、エラーは ``<error-variable>`` に格納され、``<out-var>`` にはエラーが発生した時点までのパス（関連するパスがない場合は単に ``NOTFOUND``）が付いた ``<member|index>-[<member|index>...]-NOTFOUND`` が格納される。
  エラーが発生したが、``ERROR_VARIABLE`` オプションが指定されていない場合は致命的なエラーになる。
  エラーが発生しなかった場合、``<error-variable>`` には ``NOTFOUND`` が格納される。

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         GET <json-string> <member|index> [<member|index> ...])
  :target: JSON GET

  ``<member|index> ...`` のリストで指定された場所にある ``<json-string>`` から要素を一つ取得して ``<out-var>`` に格納します。
  配列とオブジェクトの要素は JSON 文字列になります。
  論理型の要素は ``ON`` または ``OFF`` のいずれかになります。
  Null の要素は空の文字列になります。
  数値型と文字列型の要素はすべて文字列になります。

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         TYPE <json-string> <member|index> [<member|index> ...])
  :target: JSON TYPE

  ``<member|index> ...`` のリストで指定された場所にある ``<json-string>`` から要素の型を一つ取得して ``<out-var>`` に格納します。
  要素の型は ``NULL``、``NUMBER``、``STRING``、``BOOLEAN``、``ARRAY`` または ``OBJECT`` のいずれかです。

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-var>]
         MEMBER <json-string>
         [<member|index> ...] <index>)
  :target: JSON MEMBER

  ``<member|index> ...`` のリストで指定された場所にある ``<json-string>`` 内で ``<index>`` 番目のメンバの名前を取得して ``<out-var>`` に格納します。
  オブジェクト型の要素が必要になります。

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         LENGTH <json-string> [<member|index> ...])
  :target: JSON LENGTH

  ``<member|index> ...`` のリストで指定された場所にある ``<json-string>`` から要素の長さを取得して ``<out-var>`` に格納します。
  配列またはオブジェクト型の要素が必要になります。

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         REMOVE <json-string> <member|index> [<member|index> ...])
  :target: JSON REMOVE

  ``<member|index> ...`` のリストで指定された場所にある ``<json-string>`` から要素を一つ削除します。
  任意の要素を削除した JSON 文字列を ``<out-var>`` に格納します。

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-variable>]
         SET <json-string> <member|index> [<member|index> ...] <value>)
  :target: JSON SET

  ``<member|index> ...`` のリストで指定された場所にある ``<json-string>`` 内の要素の値を ``<value>`` にします。
  ``<value>`` には有効な JSON オブジェクトを指定して下さい。
  もし ``<json-string>`` が配列の場合、この配列のサイズ以上の数値を ``<member|index>`` で使用することで、``<value>`` を配列の末尾に追加できます。

.. signature::
  string(JSON <out-var> [ERROR_VARIABLE <error-var>]
         EQUAL <json-string1> <json-string2>)
  :target: JSON EQUAL

  ``<json-string1>`` と ``<json-string2>`` の 2つの JSON オブジェクトが等しいかどうか比較します。
  ``<json-string1>`` と ``<json-string2>`` には有効な JSON オブジェクトを指定して下さい。
  これら二つの JSON オブジェクトを等しいとみなした場合、``<out-var>`` には true 値を格納し、それ以外は false 値を格納します。
