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

  定数の ``<constant>`` が ``1`` または ``ON`` または ``YES`` または ``TRUE`` または ``Y``、あるいは ``0`` 以外の数値（浮動小数点）の場合は「真」である。
  ``<constant>`` が ``0`` または ``OFF`` または ``NO`` または ``FALSE`` または ``N`` または ``IGNORE`` または ``NOTFOUND``、空の文字列、あるいは末尾が ``-NOTFOUND`` で終わっている場合は「偽」である。
  ``<constant>`` の文字列は大小文字を区別しない。
  引数がこれらの ``<constant>`` のいずれにも該当しない場合は、変数または文字列（詳細は「`変数の自動展開`_」を参照のこと）として扱われ、次のルールのいずれかが適用される。

.. signature:: if(<variable>)
  :target: variable

  変数 ``<variable>`` に「偽」と評価されない定数がセットされている場合は「真」である。
  それ以外は、``<variable>`` に何もセットされていない場合を含め常に「偽」である。
  マクロで使う引数（``ARGN`` や ``ARGC`` や ``ARGV`` や ``ARGV0`` の類）は変数ではない点に注意すること。
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

  * マクロで使う引数（``ARGN`` や ``ARGC`` や ``ARGV`` や ``ARGV0`` の類）は変数ではない。
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

  ``<string>`` または ``<variable>`` の値がディクショナリ順に右辺の ``<string>`` または ``<variable>`` の値より小さい場合は「真」である。

.. signature:: if(<variable|string> STRGREATER <variable|string>)
  :target: STRGREATER

  ``<string>`` または ``<variable>`` の値がディクショナリ順に右辺の ``<string>`` または ``<variable>`` の値より大きい場合は「真」である。

.. signature:: if(<variable|string> STREQUAL <variable|string>)
  :target: STREQUAL

  ``<string>`` または ``<variable>`` の値がディクショナリ順に右辺の ``<string>`` または ``<variable>`` の値と等しい場合は「真」である。

.. signature:: if(<variable|string> STRLESS_EQUAL <variable|string>)
  :target: STRLESS_EQUAL

  .. versionadded:: 3.7

  ``<string>`` または ``<variable>`` の値がディクショナリ順に右辺の ``<string>`` または ``<variable>`` の値以下の場合は「真」である。

.. signature:: if(<variable|string> STRGREATER_EQUAL <variable|string>)
  :target: STRGREATER_EQUAL

  .. versionadded:: 3.7

  ``<string>`` または ``<variable>`` の値がディクショナリ順に右辺の ``<string>`` または ``<variable>`` の値以上の場合は「真」である。

バージョンの比較
""""""""""""""""

.. signature:: if(<variable|string> VERSION_LESS <variable|string>)
  :target: VERSION_LESS

  左辺と右辺の「バージョン」を構成する要素（書式は ``major.minor[.patch[.tweak]]`` 形式で、省略した要素は ``0`` として扱う）ごとに整数値の番号を比較し、左辺の ``<variable>`` または ``<version>`` の値が右辺の値よりも小さい場合は「真」である。
  整数値の要素を持たないバージョンや、バージョンの末尾が整数値ではない場合、そのような要素を比較する時点でバージョンの文字列から切り捨てる。

.. signature:: if(<variable|string> VERSION_GREATER <variable|string>)
  :target: VERSION_GREATER

  左辺と右辺の「バージョン」を構成する要素（書式は ``major.minor[.patch[.tweak]]`` 形式で、省略した要素は ``0`` として扱う）ごとに整数値の番号を比較し、左辺の ``<variable>`` または ``<version>`` の値が右辺の値よりも大きい場合は「真」である。
  整数値の要素を持たないバージョンや、バージョンの末尾が整数値ではない場合、そのような要素を比較する時点でバージョンの文字列から切り捨てる。

.. signature:: if(<variable|string> VERSION_EQUAL <variable|string>)
  :target: VERSION_EQUAL

  左辺と右辺の「バージョン」を構成する要素（書式は ``major.minor[.patch[.tweak]]`` 形式で、省略した要素は ``0`` として扱う）ごとに整数値の番号を比較し、左辺の ``<variable>`` または ``<version>`` の値が右辺の値と等しい場合は「真」である。
  整数値の要素を持たないバージョンや、バージョンの末尾が整数値ではない場合、そのような要素を比較する時点でバージョンの文字列から切り捨てる。

.. signature:: if(<variable|string> VERSION_LESS_EQUAL <variable|string>)
  :target: VERSION_LESS_EQUAL

  .. versionadded:: 3.7

  左辺と右辺の「バージョン」を構成する要素（書式は ``major.minor[.patch[.tweak]]`` 形式で、省略した要素は ``0`` として扱う）ごとに整数値の番号を比較し、左辺の ``<variable>`` または ``<version>`` の値が右辺の値以下の場合は「真」である。
  整数値の要素を持たないバージョンや、バージョンの末尾が整数値ではない場合、そのような要素を比較する時点でバージョンの文字列から切り捨てる。

.. signature:: if(<variable|string> VERSION_GREATER_EQUAL <variable|string>)
  :target: VERSION_GREATER_EQUAL

  .. versionadded:: 3.7

  左辺と右辺の「バージョン」を構成する要素（書式は ``major.minor[.patch[.tweak]]`` 形式で、省略した要素は ``0`` として扱う）ごとに整数値の番号を比較し、左辺の ``<variable>`` または ``<version>`` の値が右辺の値以上の場合は「真」である。
  整数値の要素を持たないバージョンや、バージョンの末尾が整数値ではない場合、そのような要素を比較する時点でバージョンの文字列から切り捨てる。

パスの比較
""""""""""

.. signature:: if(<variable|string> PATH_EQUAL <variable|string>)
  :target: PATH_EQUAL

  .. versionadded:: 3.24

  左辺と右辺のパスをそれを構成する要素ごとに比較する。
  両方のパスの全ての要素が一致する場合にのみ、二つのパスが同等であるとして「真」を返す。
  連続するパスの区切り文字（``directory-separator``）は、１つにまとめられるが、バックスラッシュ（``\``）はスラッシュ（``/``）にはまとめられない点に注意すること。
  :ref:`パスの正規化 <Normalization>` は行わない。

  要素ごとの比較は、連続するパスの区切り文字（``directory-separator``）を正しく処理できるので、文字列ごとの比較よりも正確である。
  次の例では、``PATH_EQUAL`` で比較すると「真」を返すが、``STREQUAL`` で比較すると「偽」と評価される：

  .. code-block:: cmake

    # 結果は「真」
    if ("/a//b/c" PATH_EQUAL "/a/b/c")
       ...
    endif()

    # 結果は「偽」
    if ("/a//b/c" STREQUAL "/a/b/c")
       ...
    endif()

  詳細は :ref:`cmake_path(COMPARE) <Path COMPARE>` コマンドの説明を参照のこと。

変数の自動展開
^^^^^^^^^^^^^^

この ``if`` コマンドは ``${}`` 変数の導入よりも前（CMake プロジェクトの黎明期）に実装されたものであり、便宜上はこれまでの解説にあるとおり、「引数」として渡された変数を評価します。
ただし ``${}`` 変数の評価は、``if`` コマンドがその変数を **受け取る前に自動的に展開されている** という点に留意して下さい。
したがって、次のようなコードの場合：

.. code-block:: cmake

 set(var1 OFF)
 set(var2 "var1")
 if(${var2})

``if`` コマンドには次のように見えます：

.. code-block:: cmake

  if(var1)

そして ``if(<variable>)`` の比較として評価されます。
この場合 ``var`` は ``OFF`` なので結果は「偽」になります。
ただし、このコードから ``${}`` を取り去ると、``if`` コマンドは次のように見えます：

.. code-block:: cmake

  if(var2)

すなわち ``var2`` は定数の ``"var1"`` として定義されているので「真」を返します。

変数の自動展開は、「`条件式の構文`_」が ``<variable|string>`` を受け入れる場合、常にいろいろなケースで適用されます（FIXME: 意味不明）：

* まず最初に `MATCHES`_ の左辺の引数が既に定義されている変数であるかチェックする。
  定義されている変数だったら、その変数の値を使用する。
  定義されていなければ、その引数を値としてそのまま使用する。

* `MATCHES`_ の左辺に引数が無い場合は、エラーにしないで、「偽」を返す。

* `LESS`_ や `GREATER`_ や `EQUAL`_ や `LESS_EQUAL`_ や `GREATER_EQUAL`_ の左辺と右辺の引数が、それぞれ定義されている変数であるか別個にチェックする。
  定義されている変数だったら、それら変数の値を使用する。
  定義されていなければ、その引数を値としてそのまま使用する。

* `STRLESS`_ や `STRGREATER`_ や `STREQUAL`_ や `STRLESS_EQUAL`_, や `STRGREATER_EQUAL`_ の左辺と右辺の引数が、それぞれ定義されている変数であるか別個にチェックする。
  定義されている変数だったら、それら変数の値を使用する。
  定義されていなければ、その引数を値としてそのまま使用する。

* `VERSION_LESS`_ や `VERSION_GREATER`_ や `VERSION_EQUAL`_ や `VERSION_LESS_EQUAL`_ や `VERSION_GREATER_EQUAL`_ の左辺と右辺の引数が、それぞれ定義されている変数であるか別個にチェックする。
  定義されている変数だったら、それら変数の値を使用する。
  定義されていなければ、その引数を値としてそのまま使用する。

* `NOT`_ の右辺の引数が論理型の定数かどうかをチェックする。
  論理型の定数ならば、その値をそのまま使用する。
  論理型の定数でなければ、その引数を変数とみなし、その変数の値を使用する。

* `AND`_ や `OR`_ の左辺と右辺の引数が論理型の定数かどうかを別個にチェックする。
  論理型の定数ならば、その値をそのまま使用する。
  論理型の定数でなければ、その引数を変数とみなし、その変数の値を使用する。

.. versionchanged:: 3.1
  評価が曖昧にならないようにするため、変数名やキーワードを :ref:`Quoted Argument` や :ref:`Bracket Argument` として指定することができるようになった。
  これにより、引用符またはカッコで囲んだ変数やキーワードは「文字列」として解釈されるので、逆参照されたり別の値に解釈されることはない。
  :policy:`CMP0054` のポリシーも参照のこと。

:ref:`CMake の環境変数 <CMake Language Environment Variables>` やキャッシュ変数の :ref:`Variable References` に対して自動展開は行いません。
「`条件式の構文`_」が ``<variable|string>`` を受け入れる場合は、その値を参照する場合は ``$ENV{<name>}`` や ``$CACHE{<name>}`` を使う必要があります。

参考情報
^^^^^^^^

* :command:`else`
* :command:`elseif`
* :command:`endif`
