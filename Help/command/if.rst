if
--

コマンドのグループを条件付きで実行する。

概要
^^^^

.. code-block:: cmake

  if(<condition>)
    <commands>
  elseif(<condition>) # オプションのブロック（複数指定が可能）
    <commands>
  else()              # オプションのブロック
    <commands>
  endif()

このドキュメントにある「`条件式の構文`_」に従って、``if`` 句の ``<condition>`` を評価します。
評価した結果が ``TRUE`` の場合、``if`` 句の中にある ``<commands>`` ブロックが実行されます。
それ以外はオプションである ``elseif`` 句が同じように処理されます。
そして最後に ``<condition>`` が ``TRUE`` でなければ、オプションである ``else`` 句の ``<commands>`` が実行されます。

従来どおり、:command:`else` と :command:`endif` コマンドはオプションで ``<condition>`` を引数として受け取れます。
その場合、最初の ``if`` コマンドの引数をそのまま繰り返すようにして下さい。

.. _`Condition Syntax`:

条件式の構文
^^^^^^^^^^^^

次に示す構文ルールは、``if`` や :command:`endif` 句、そして :command:`while` 句の ``<condition>`` に適用されます。

複合化した条件式は、次の優先順位で評価されます：

1. カッコ（`Parentheses`_）

2. 単項テスト（`EXISTS`_、`COMMAND`_、`DEFINED`_）

3. 二単テスト（`EQUAL`_、`LESS`_、`LESS_EQUAL`_、`GREATER`_、`GREATER_EQUAL`_,、`STREQUAL`_、`STRLESS`_、`STRLESS_EQUAL`_、`STRGREATER`_、`STRGREATER_EQUAL`_、`VERSION_EQUAL`_、`VERSION_LESS`_、`VERSION_LESS_EQUAL`_、`VERSION_GREATER`_、`VERSION_GREATER_EQUAL`_、`PATH_EQUAL`_、`MATCHES`_）

4. 単項論理演算子の `NOT`_

5. 二項論理演算子の `AND`_ と `OR`_ （左から右へ最短で評価していく）

基本的な条件式
""""""""""""""

.. signature:: if(<constant>)
  :target: constant

  定数``<constant>`` が ``1`` または ``ON`` または ``YES`` または ``TRUE`` または ``Y``、あるいは ``0`` 以外の数値（浮動小数点）の場合は「真」である。
  ``<constant>`` が ``0`` または ``OFF`` または ``NO`` または ``FALSE`` または ``N`` または ``IGNORE`` または ``NOTFOUND``、空の文字列、あるいは末尾が ``-NOTFOUND`` で終わっている場合は「偽」である。
  ``<constant>`` の文字列は大小文字を区別しない。
  引数がこれらの ``<constant>`` のいずれにも該当しない場合は、変数または文字列（詳細は「`変数の展開`_」を参照のこと）として扱われ、次のルールのいずれかが適用される。

.. signature:: if(<variable>)
  :target: variable

  変数 ``<variable>`` に「偽」と評価されない定数がセットされている場合は「真」である。
  それ以外は、``<variable>`` に何もセットされていない場合を含め常に「偽」である。
  マクロ引数（:command:`macro()`）は変数ではない点に注意すること。
  また :ref:`CMake の環境変数 <CMake Language Environment Variables>` もこの方法ではテストできない（例えば ``if(ENV{some_var})`` は常に「偽」と評価される）。

.. signature:: if(<string>)
  :target: string

  引用符で囲まれた文字列 ``<string>`` は、次の場合を除いて、常に「偽」である。

  * ``<string>`` は「真」と評価される定数である。
  * ポリシーの :policy:`CMP0054` に従って ``<string>`` に ``NEW`` がセットされていないか、または ``<variable>`` が :policy:`CMP0054` の挙動に影響を与える変数名である。

論理式の評価
""""""""""""

.. signature:: if(NOT <condition>)

  ``<condition>`` が「真」でなければ「真」である。

.. signature:: if(<cond1> AND <cond2>)
  :target: AND

  ``<cond1>`` と ``<cond2>`` が共に「真」の場合は「真」である。

.. signature:: if(<cond1> OR <cond2>)
  :target: OR

  ``<cond1>`` と ``<cond2>`` のどちらかが「真」の場合は「真」である。

.. signature:: if((condition) AND (condition OR (condition)))
  :target: parentheses

  まずカッコ内の ``<condition>`` が最初に評価され、次に残りの ``<condition>`` が評価される。
  ネストされたカッコがある場合は、最も内側のカッコの中にある ``<condition>`` が、カッコを含む条件の一部として評価される。

存在するかどうかのテスト
""""""""""""""""""""""""

.. signature:: if(COMMAND <command-name>)

  ``<command-name>`` が CMake から呼び出すことが可能なコマンド、マクロ、あるいは関数の場合は「真」である。

.. signature:: if(POLICY <policy-id>)

  ``<policy-id>`` が既存のポリシー（``CMP<NNNN>`` 形式）の一つである場合は「真」である。

.. signature:: if(TARGET <target-name>)

  ``<target-name>`` が、既に（任意のディレクトリで）CMake から呼び出された :command:`add_executable` または :command:`add_library` または :command:`add_custom_target` コマンドで作成・追加された論理ターゲットである場合は「真」である。

.. signature:: if(TEST <test-name>)

  .. versionadded:: 3.3

  ``<test-name>`` が :command:`add_test` コマンドで作成・追加されたテスト名である場合は「真」である。

.. signature:: if(DEFINED <name>|CACHE{<name>}|ENV{<name>})

  ``<name>`` という名前の変数やキャッシュ変数、または環境変数が定義されている場合は「真」である。
  変数の値はテストしない。
  次の注意事項に留意すること：

  * マクロ引数（:command:`macro()`）は変数ではない。
  * ``<name>`` が **（キャッシュ変数ではない）通常の変数であることを直接テストすることはできない**。
    ``if(DEFINED someName)`` という式で ``someName`` というキャッシュ変数または通常の変数が定義されている場合は常に「真」である。
    対して ``if(DEFINED CACHE{someName})`` という式では、``someName`` というキャッシュ変数が定義されている場合にのみ「真」である。
    **通常の変数が定義されているかどうか** を知る必要がある場合は、両方の条件式をテストする必要がある： ``if(DEFINED someName AND NOT DEFINED CACHE{someName})``

 .. versionadded:: 3.14
  ``CACHE{<name>}`` のテストを追加した。

.. signature:: if(<variable|string> IN_LIST <variable>)
  :target: IN_LIST

  .. versionadded:: 3.3

  ``<variable>`` または ``<string>`` が :ref:`リスト <CMake Language Lists>` 型の名前付き変数に含まれている場合は「真」である。

ファイルのテスト
""""""""""""""""

.. signature:: if(EXISTS <path-to-file-or-directory>)

  ``<path-to-file-or-directory>`` というファイルまたはディレクトリが存在し、CMake で読み取りが可能な場合は「真」である。
  これは絶対パスで指定した場合にのみ明確にテストできる（すなわち、先頭にある ``~/`` はホームディレクトリとは解釈されず、相対パスとみなされる）。
  テストする前にシンボリックリンクは解決する（すなわち、``<path-to-file-or-directory>`` がシンボリックリンクの場合、シンボリックリンクのターゲットが存在している場合は「真」である）。

  ``<path-to-file-or-directory>`` が空文字の場合は「偽」である。

.. signature:: if(<file1> IS_NEWER_THAN <file2>)
  :target: IS_NEWER_THAN

  ``<file1>`` が ``<file2>`` よりも新しいか、または二つのファイルのうちいずれかが存在していない場合は「真」である。
  これは絶対パスで指定した場合にのみ明確にテストできる。
  ``<file1>`` と ``<file2>`` のタイムスタンプが全く同じである場合は「真」を返すので、この評価に依存するビルド操作は同時に発生する。
  このケースは、``<file1>`` と ``<file2>`` の両方に同じファイル名を渡した場合も含まれる。

.. signature:: if(IS_DIRECTORY <path>)

  ``<path>`` がディレクトリの場合は「真」である。
  これは絶対パスで指定した場合にのみ明確にテストできる。

  ``<path>`` が空文字の場合は「偽」である。

.. signature:: if(IS_SYMLINK <path>)

  ``<path>`` がシンボリックリンクの場合は「真」である。
  これは絶対パスで指定した場合にのみ明確にテストできる。

.. signature:: if(IS_ABSOLUTE <path>)

  ``<path>`` が絶対パスの場合は「真」である。
  次のような特殊なケースに留意すること：

  * ``<path>`` が空文字の場合は「偽」である。
  * ホストが Windows 系プラットフォームの場合、ドライブ文字とコロンからなるパス名（例えば ``C:``）やスラッシュやバックスラッシュで始まるパス名は全て「真」である。
    これは、例えば ``C:no\base\dir`` のように、パス名でドライブ文字以外の部分が相対パスであっても「真」として評価されることを意味する。
  * ホストが Windows 系以外のプラットフォームの場合、先頭がチルダ（``~``）で始まる ``<path>`` は全て「真」である。

比較式の評価
""""""""""""

.. signature:: if(<variable|string> MATCHES <regex>)
  :target: MATCHES

  ``<string>`` または ``<variable>`` の値が正規表現の ``<regex>`` にマッチする場合は「真」である。
  利用可能な正規表現については「:ref:`正規表現の仕様 <Regex Specification>`」を参照のこと。

  .. versionadded:: 3.9
   正規表現のグループ ``()`` は :variable:`CMAKE_MATCH_<n>` 変数で補足し参照できるようになった。

.. signature:: if(<variable|string> LESS <variable|string>)
  :target: LESS

  ``<string>`` または ``<variable>`` の値が実数（C言語の ``double`` 型など）として解析され、右辺の値よりも小さい場合は「真」である。

.. signature:: if(<variable|string> GREATER <variable|string>)
  :target: GREATER

  ``<string>`` または ``<variable>`` の値が実数（C言語の ``double`` 型など）として解析され、右辺の値よりも大きい場合は「真」である。

.. signature:: if(<variable|string> EQUAL <variable|string>)
  :target: EQUAL

  ``<string>`` または ``<variable>`` の値が実数（C言語の ``double`` 型など）として解析され、右辺の値と等しい場合は「真」である。

.. signature:: if(<variable|string> LESS_EQUAL <variable|string>)
  :target: LESS_EQUAL

  .. versionadded:: 3.7

  ``<string>`` または ``<variable>`` の値が実数（C言語の ``double`` 型など）として解析され、右辺の値以下である場合は「真」である。

.. signature:: if(<variable|string> GREATER_EQUAL <variable|string>)
  :target: GREATER_EQUAL

  .. versionadded:: 3.7

  ``<string>`` または ``<variable>`` の値が実数（C言語の ``double`` 型など）として解析され、右辺の値以上である場合は「真」である。

.. signature:: if(<variable|string> STRLESS <variable|string>)
  :target: STRLESS

  ``<string>`` または ``<variable>`` の値がディクショナリ順に右辺の ``<string>`` または ``<variable>`` より小さい場合は「真」である。

.. signature:: if(<variable|string> STRGREATER <variable|string>)
  :target: STRGREATER

  True if the given string or variable's value is lexicographically greater than the string or variable on the right.

.. signature:: if(<variable|string> STREQUAL <variable|string>)
  :target: STREQUAL

  True if the given string or variable's value is lexicographically equal to the string or variable on the right.

.. signature:: if(<variable|string> STRLESS_EQUAL <variable|string>)
  :target: STRLESS_EQUAL

  .. versionadded:: 3.7

  True if the given string or variable's value is lexicographically less than or equal to the string or variable on the right.

.. signature:: if(<variable|string> STRGREATER_EQUAL <variable|string>)
  :target: STRGREATER_EQUAL

  .. versionadded:: 3.7

  True if the given string or variable's value is lexicographically greater than or equal to the string or variable on the right.

バージョンの比較
""""""""""""""""

.. signature:: if(<variable|string> VERSION_LESS <variable|string>)
  :target: VERSION_LESS

  Component-wise integer version number comparison (version format is ``major[.minor[.patch[.tweak]]]``, omitted components are treated as zero).
  Any non-integer version component or non-integer trailing part of a version component effectively truncates the string at that point.

.. signature:: if(<variable|string> VERSION_GREATER <variable|string>)
  :target: VERSION_GREATER

  Component-wise integer version number comparison (version format is ``major[.minor[.patch[.tweak]]]``, omitted components are treated as zero).
  Any non-integer version component or non-integer trailing part of a version component effectively truncates the string at that point.

.. signature:: if(<variable|string> VERSION_EQUAL <variable|string>)
  :target: VERSION_EQUAL

  Component-wise integer version number comparison (version format is
  ``major[.minor[.patch[.tweak]]]``, omitted components are treated as zero).
  Any non-integer version component or non-integer trailing part of a version
  component effectively truncates the string at that point.

.. signature:: if(<variable|string> VERSION_LESS_EQUAL <variable|string>)
  :target: VERSION_LESS_EQUAL

  .. versionadded:: 3.7

  Component-wise integer version number comparison (version format is
  ``major[.minor[.patch[.tweak]]]``, omitted components are treated as zero).
  Any non-integer version component or non-integer trailing part of a version
  component effectively truncates the string at that point.

.. signature:: if(<variable|string> VERSION_GREATER_EQUAL <variable|string>)
  :target: VERSION_GREATER_EQUAL

  .. versionadded:: 3.7

  Component-wise integer version number comparison (version format is
  ``major[.minor[.patch[.tweak]]]``, omitted components are treated as zero).
  Any non-integer version component or non-integer trailing part of a version
  component effectively truncates the string at that point.

パスの比較
""""""""""

.. signature:: if(<variable|string> PATH_EQUAL <variable|string>)
  :target: PATH_EQUAL

  .. versionadded:: 3.24

  Compares the two paths component-by-component.  Only if every component of
  both paths match will the two paths compare equal.  Multiple path separators
  are effectively collapsed into a single separator, but note that backslashes
  are not converted to forward slashes.  No other
  :ref:`path normalization <Normalization>` is performed.

  Component-wise comparison is superior to string-based comparison due to the
  handling of multiple path separators.  In the following example, the
  expression evaluates to true using ``PATH_EQUAL``, but false with
  ``STREQUAL``:

  .. code-block:: cmake

    # comparison is TRUE
    if ("/a//b/c" PATH_EQUAL "/a/b/c")
       ...
    endif()

    # comparison is FALSE
    if ("/a//b/c" STREQUAL "/a/b/c")
       ...
    endif()

  See :ref:`cmake_path(COMPARE) <Path COMPARE>` for more details.

変数の展開
^^^^^^^^^^

The if command was written very early in CMake's history, predating
the ``${}`` variable evaluation syntax, and for convenience evaluates
variables named by its arguments as shown in the above signatures.
Note that normal variable evaluation with ``${}`` applies before the if
command even receives the arguments.  Therefore code like

.. code-block:: cmake

 set(var1 OFF)
 set(var2 "var1")
 if(${var2})

appears to the if command as

.. code-block:: cmake

  if(var1)

and is evaluated according to the ``if(<variable>)`` case documented
above.  The result is ``OFF`` which is false.  However, if we remove the
``${}`` from the example then the command sees

.. code-block:: cmake

  if(var2)

which is true because ``var2`` is defined to ``var1`` which is not a false
constant.

Automatic evaluation applies in the other cases whenever the
above-documented condition syntax accepts ``<variable|string>``:

* The left hand argument to `MATCHES`_ is first checked to see if it is
  a defined variable.  If so, the variable's value is used, otherwise the
  original value is used.

* If the left hand argument to `MATCHES`_ is missing it returns false
  without error

* Both left and right hand arguments to `LESS`_, `GREATER`_, `EQUAL`_,
  `LESS_EQUAL`_, and `GREATER_EQUAL`_, are independently tested to see if
  they are defined variables.  If so, their defined values are used otherwise
  the original value is used.

* Both left and right hand arguments to `STRLESS`_, `STRGREATER`_,
  `STREQUAL`_, `STRLESS_EQUAL`_, and `STRGREATER_EQUAL`_ are independently
  tested to see if they are defined variables.  If so, their defined values are
  used otherwise the original value is used.

* Both left and right hand arguments to `VERSION_LESS`_,
  `VERSION_GREATER`_, `VERSION_EQUAL`_, `VERSION_LESS_EQUAL`_, and
  `VERSION_GREATER_EQUAL`_ are independently tested to see if they are defined
  variables.  If so, their defined values are used otherwise the original value
  is used.

* The right hand argument to `NOT`_ is tested to see if it is a boolean
  constant.  If so, the value is used, otherwise it is assumed to be a
  variable and it is dereferenced.

* The left and right hand arguments to `AND`_ and `OR`_ are independently
  tested to see if they are boolean constants.  If so, they are used as
  such, otherwise they are assumed to be variables and are dereferenced.

.. versionchanged:: 3.1
  To prevent ambiguity, potential variable or keyword names can be
  specified in a :ref:`Quoted Argument` or a :ref:`Bracket Argument`.
  A quoted or bracketed variable or keyword will be interpreted as a
  string and not dereferenced or interpreted.
  See policy :policy:`CMP0054`.

There is no automatic evaluation for environment or cache
:ref:`Variable References`.  Their values must be referenced as
``$ENV{<name>}`` or ``$CACHE{<name>}`` wherever the above-documented
condition syntax accepts ``<variable|string>``.

参考情報
^^^^^^^^

* :command:`else`
* :command:`elseif`
* :command:`endif`
