cmake_path
----------

.. versionadded:: 3.20

「**CMake スタイルのパス**」を操作するコマンド。
CMake スタイルのパスを構成している要素だけ処理し、実際のファイルシステムとのやりとりは行なわない。
そのため CMake スタイルのパスは「実際には存在しないパス」とか「現在のファイルシステムやプットフォームに存在することが許されないパス」を表す場合がある。
これに対し、ファイルシステムとやり取りする操作については :command:`file` コマンドを参照のこと。

.. note::

  ``cmake_path`` コマンドはターゲットではなく、ビルドシステム（すなわちホストのプラットフォーム）のスタイルでパスを処理します。
  クロス・コンパイルで、ホストのプラットフォームでは扱えない要素（たとえばホストが Windows でない時のドライブ名など）がパス含まれていたら、その結果は不明です。

概要
^^^^

.. parsed-literal::

  `凡例`_

  `パスを構成する要素`_

  `パス変数を作成する`_

  `パスを正規化する`_

  `パスの要素を取得する`_
    cmake_path(`GET`_ <path-var> :ref:`ROOT_NAME <GET_ROOT_NAME>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`ROOT_DIRECTORY <GET_ROOT_DIRECTORY>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`ROOT_PATH <GET_ROOT_PATH>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`FILENAME <GET_FILENAME>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`EXTENSION <GET_EXTENSION>` [LAST_ONLY] <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`STEM <GET_STEM>` [LAST_ONLY] <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`RELATIVE_PART <GET_RELATIVE_PART>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`PARENT_PATH <GET_PARENT_PATH>` <out-var>)

  `パスの要素を照会する`_
    cmake_path(`HAS_ROOT_NAME`_ <path-var> <out-var>)
    cmake_path(`HAS_ROOT_DIRECTORY`_ <path-var> <out-var>)
    cmake_path(`HAS_ROOT_PATH`_ <path-var> <out-var>)
    cmake_path(`HAS_FILENAME`_ <path-var> <out-var>)
    cmake_path(`HAS_EXTENSION`_ <path-var> <out-var>)
    cmake_path(`HAS_STEM`_ <path-var> <out-var>)
    cmake_path(`HAS_RELATIVE_PART`_ <path-var> <out-var>)
    cmake_path(`HAS_PARENT_PATH`_ <path-var> <out-var>)
    cmake_path(`IS_ABSOLUTE`_ <path-var> <out-var>)
    cmake_path(`IS_RELATIVE`_ <path-var> <out-var>)
    cmake_path(`IS_PREFIX`_ <path-var> <input> [NORMALIZE] <out-var>)
    cmake_path(`COMPARE`_ <input1> <OP> <input2> <out-var>)

  `パスを編集する`_
    cmake_path(:ref:`SET <cmake_path-SET>` <path-var> [NORMALIZE] <input>)
    cmake_path(`APPEND`_ <path-var> [<input>...] [OUTPUT_VARIABLE <out-var>])
    cmake_path(`APPEND_STRING`_ <path-var> [<input>...] [OUTPUT_VARIABLE <out-var>])
    cmake_path(`REMOVE_FILENAME`_ <path-var> [OUTPUT_VARIABLE <out-var>])
    cmake_path(`REPLACE_FILENAME`_ <path-var> <input> [OUTPUT_VARIABLE <out-var>])
    cmake_path(`REMOVE_EXTENSION`_ <path-var> [LAST_ONLY] [OUTPUT_VARIABLE <out-var>])
    cmake_path(`REPLACE_EXTENSION`_ <path-var> [LAST_ONLY] <input> [OUTPUT_VARIABLE <out-var>])

  `パスを変換する`_
    cmake_path(`NORMAL_PATH`_ <path-var> [OUTPUT_VARIABLE <out-var>])
    cmake_path(`RELATIVE_PATH`_ <path-var> [BASE_DIRECTORY <input>] [OUTPUT_VARIABLE <out-var>])
    cmake_path(`ABSOLUTE_PATH`_ <path-var> [BASE_DIRECTORY <input>] [NORMALIZE] [OUTPUT_VARIABLE <out-var>])

  `ネィティブ・スタイルに変換する`_
    cmake_path(`NATIVE_PATH`_ <path-var> [NORMALIZE] <out-var>)
    cmake_path(`CONVERT`_ <input> `TO_CMAKE_PATH_LIST`_ <out-var> [NORMALIZE])
    cmake_path(`CONVERT`_ <input> `TO_NATIVE_PATH_LIST`_ <out-var> [NORMALIZE])

  `ハッシュ値`_
    cmake_path(`HASH`_ <path-var> <out-var>)

.. _Conversions:

凡例
^^^^

このコマンドは次の凡例に準じます：

``<path-var>``
  変数名を表す。
  入力として ``<path-var>`` を期待するコマンドの場合、必ず変数が存在し、その変数が単一のパスを保持していることが期待される。

``<input>``
  コマンドに応じて、パスだったり、パスの一部だったり、あるいは特別な区切り文字を含んだ複数のパスからなる文字列リテラルだったりする。
  それぞれどのように解釈されるかについては、関連するコマンドの説明を参照のこと。

``<input>...``
  0個以上の文字列リテラルの引数を表す。

``<out-var>``
  コマンドの結果を保持する変数名を表す。


.. _Path Structure And Terminology:

パスを構成する要素
^^^^^^^^^^^^^^^^^^

CMake スタイルのパスは次のような構造を持ちます（全ての要素はオプションですが、いくつかの制限があります）：

::

  root-name root-directory-separator (item-name directory-separator)* filename

``root-name``
  複数の root ディレクトリ（たとえば ``"C:"`` や ``"//myserver"`` など）があるファイルシステム上での root を表す（オプション）。

``root-directory-separator``
  ディレクトリの区切り文字。
  この文字が含まれているパスは絶対パスであることを示す。
  この区切り文字が無く、先頭の要素が ``root-name`` 以外の ``item-name`` の場合、そのパスは相対パスとして扱われる。

``item-name``
  ディレクトリの区切り文字ではない文字列リテラル。
  この名前は一個のファイル、一個のハード・リンク、一個のシンボリックリンク、あるいは一個のディレクトリを表す。
  次のような特殊なケースが二つある：

    * 一個のドット文字（"``.``"）を含む ``item-name`` は現在のディレクトリを表す

    * 二個のドット文字（"``..``"）を含む ``item-name`` は親ディレクトリを表す

  上に示した ``(item-name directory-separator)*`` のパタンは、``directory-separator`` で複数の要素が区切られ、０個以上の ``item-name`` が存在できることを示す。
  なお ``()*`` の部分はパスの一部ではない。

``directory-separator``
  ディレクトリの区切り文字として認識されるのはスラッシュ文字（"``/``"）だけ。
  この文字が繰り返されても、一個のディレクトリの区切り文字として扱う。
  つまり、``/usr///////lib`` は ``/usr/lib`` と同じ。

.. _FILENAME_DEF:
.. _EXTENSION_DEF:
.. _STEM_DEF:

``filename``
  ファイル名を表す（すなわち、文字列の終端が ``directory-separator`` ではないパスのこと）。
  実のところ ``filename`` はパスの最後にある ``item-name`` なので、一個のハードリンク、一個のシンボリックリンク、または一個のディレクトリにすることも可能である。

  ``filename`` には一個の *拡張子* をつけることが可能である。
  デフォルトで、拡張子はピリオドから ``filename`` の末尾までの部分文字列として定義される。
  ``LAST_ONLY`` というキーワードを受け取るコマンドでは、``LAST_ONLY`` が拡張子でピリオドを除く部分文字列に置き換わる。

  拡張子には、次に示す例外が適用される：

    * もし ``filename`` の先頭の文字が一個のドット文字（"``.``"）だったら、拡張子の検出では無視される（たとえば ``".profile"`` は拡張子なしとして扱う）

    * もし ``filename`` が ``.`` または ``..`` だったら、拡張子なしとして扱う

  ステム（*STEM*）とは ``filename`` の拡張子よりも前にあるの部分文字列を指す。

一部のコマンドは ``root-path`` を参照します。
これは ``root-name`` と ``root-directory-separator`` を連結したもので、どちらかまたは両方を空文字にすることが可能です。
``relative-part`` は ``root-path`` を除く、絶対パス名を指します。


パス変数を作成する
^^^^^^^^^^^^^^^^^^

パス変数は、普通に :command:`set` コマンドで作成できますが、必要に応じて自動的にパスを任意の形式に変換させたい場合は、代わりに :ref:`cmake_path(SET) <cmake_path-SET>` コマンドの利用を推奨します。
また、サブコマンドの :ref:`cmake_path(APPEND) <APPEND>` は、任意の文字列を連結してパスを構築していく手段として利用できます。
次の例では、同じパスを構築する三つの方法を比較しています：

.. code-block:: cmake

  set(path1 "${CMAKE_CURRENT_SOURCE_DIR}/data")

  cmake_path(SET path2 "${CMAKE_CURRENT_SOURCE_DIR}/data")

  cmake_path(APPEND path3 "${CMAKE_CURRENT_SOURCE_DIR}" "data")

`パスを編集する`_ と `パスを変換する`_ サブコマンドは、実行結果をその場で保存することも、あるいは ``OUTPUT_VARIABLE`` というキーワードを持つ別の変数に保存することもできます。 
それ以外のサブコマンドは全て ``<out-var>`` 変数に結果を保存します。

.. _Normalization:

パスを正規化する
^^^^^^^^^^^^^^^^

一部のサブコマンドはパスの *正規化* をサポートしています。
パスを正規化するアルゴリズムは次のとおりです：

1. パスが空文字だったら正規化を停止する（空文字を正規化しても空文字です）
2. パスの中で、ディレクトリの区切り文字（"``/``"）が連続している可能性がある ``directory-separator`` を一個の ``/``  文字で置き換える（たとえば ``/a/////b  --> /a/b``）
3. パスの中で、一個のドット文字（"``.``"）とその直後の ``directory-separator`` を削除する（たとえば ``/a/./b/. --> /a/b``）
4. パスの中で、"``..``" を除く ``item-name`` の直後に ``directory-separator`` と二個のドット文字（"``..``"）が連続している部分を削除する（たとえば ``/a/b/../c --> /a/c``）
5. パスの中に ``root-directory`` がある場合は、その直後の ``..`` と ``directory-separators`` のペアを削除する（たとえば ``/../a --> /a``）
6. パスの最後の ``item-name`` が二個のドット文字（``..``） だったら、末尾の ``directory-separator`` を削除する（たとえば ``../ --> ..``）
7. この段階でパスが空文字だったら一個のドット文字（"``.``"）を追加する（``./`` の標準的な書式は ``.``）


.. _Path Decomposition:

パスの要素を取得する
^^^^^^^^^^^^^^^^^^^^

.. _GET:
.. _GET_ROOT_NAME:
.. _GET_ROOT_DIRECTORY:
.. _GET_ROOT_PATH:
.. _GET_FILENAME:
.. _GET_EXTENSION:
.. _GET_STEM:
.. _GET_RELATIVE_PART:
.. _GET_PARENT_PATH:

``GET`` サブコマンドは、引数として渡したパスからそれを構成している要素や要素のグループを取得します。
パスを構成する要素の意味については「`パスを構成する要素`_」の項を参照して下さい。

.. code-block:: cmake

  cmake_path(GET <path-var> ROOT_NAME <out-var>)
  cmake_path(GET <path-var> ROOT_DIRECTORY <out-var>)
  cmake_path(GET <path-var> ROOT_PATH <out-var>)
  cmake_path(GET <path-var> FILENAME <out-var>)
  cmake_path(GET <path-var> EXTENSION [LAST_ONLY] <out-var>)
  cmake_path(GET <path-var> STEM [LAST_ONLY] <out-var>)
  cmake_path(GET <path-var> RELATIVE_PART <out-var>)
  cmake_path(GET <path-var> PARENT_PATH <out-var>)

このコマンドで要求した要素がパスの中に存在しない場合は空の文字列を ``<out-var>`` に格納します。
たとえば ``root-name`` の概念があるのは Windows のシステムだけなので、ホスト・マシンが Windows 以外のシステムの場合、``ROOT_NAME`` サブコマンドは常に空の文字列を返します。

``PARENT_PATH`` サブコマンドは、もし `HAS_RELATIVE_PART`_ サブコマンドが ``FALSE`` を返す場合、その結果は指定した ``<path-var>`` と同じ（コピー）です。
なお 「**CMake スタイルの root ディレクトリ**」の定義は「親ディレクトリがある」＋「親ディレクトリは自分自身である」ということに留意しておいて下さい。
これに対して `HAS_RELATIVE_PART`_ サブコマンドが ``TRUE`` を返す場合、その結果は基本的に ``<path-var>`` の末尾の要素を削除したものになります。

ROOT 系サブコマンドの例
"""""""""""""""""""""""

.. code-block:: cmake

  set(path "c:/a")

  cmake_path(GET path ROOT_NAME rootName)
  cmake_path(GET path ROOT_DIRECTORY rootDir)
  cmake_path(GET path ROOT_PATH rootPath)

  message("Root name is \"${rootName}\"")
  message("Root directory is \"${rootDir}\"")
  message("Root path is \"${rootPath}\"")

::

  Root name is "c:"
  Root directory is "/"
  Root path is "c:/"

FILENAME 系サブコマンドの例
"""""""""""""""""""""""""""

.. code-block:: cmake

  set(path "/a/b")
  cmake_path(GET path FILENAME filename)
  message("First filename is \"${filename}\"")

  # 終端の '/' はファイル名が空であることを示す
  set(path "/a/b/")
  cmake_path(GET path FILENAME filename)
  message("Second filename is \"${filename}\"")

::

  First filename is "b"
  Second filename is ""

拡張子とステムの例
""""""""""""""""""

.. code-block:: cmake

  set(path "name.ext1.ext2")

  cmake_path(GET path EXTENSION fullExt)
  cmake_path(GET path STEM fullStem)
  message("Full extension is \"${fullExt}\"")
  message("Full stem is \"${fullStem}\"")

  # LAST_ONLY オプションの効果
  cmake_path(GET path EXTENSION LAST_ONLY lastExt)
  cmake_path(GET path STEM LAST_ONLY lastStem)
  message("Last extension is \"${lastExt}\"")
  message("Last stem is \"${lastStem}\"")

  # 特殊なケース
  set(dotPath "/a/.")
  set(dotDotPath "/a/..")
  set(someMorePath "/a/.some.more")
  cmake_path(GET dotPath EXTENSION dotExt)
  cmake_path(GET dotPath STEM dotStem)
  cmake_path(GET dotDotPath EXTENSION dotDotExt)
  cmake_path(GET dotDotPath STEM dotDotStem)
  cmake_path(GET dotMorePath EXTENSION someMoreExt)
  cmake_path(GET dotMorePath STEM someMoreStem)
  message("Dot extension is \"${dotExt}\"")
  message("Dot stem is \"${dotStem}\"")
  message("Dot-dot extension is \"${dotDotExt}\"")
  message("Dot-dot stem is \"${dotDotStem}\"")
  message(".some.more extension is \"${someMoreExt}\"")
  message(".some.more stem is \"${someMoreStem}\"")

::

  Full extension is ".ext1.ext2"
  Full stem is "name"
  Last extension is ".ext2"
  Last stem is "name.ext1"
  Dot extension is ""
  Dot stem is "."
  Dot-dot extension is ""
  Dot-dot stem is ".."
  .some.more extension is ".more"
  .some.more stem is ".some"

相対パスの例
""""""""""""

.. code-block:: cmake

  set(path "c:/a/b")
  cmake_path(GET path RELATIVE_PART result)
  message("Relative part is \"${result}\"")

  set(path "c/d")
  cmake_path(GET path RELATIVE_PART result)
  message("Relative part is \"${result}\"")

  set(path "/")
  cmake_path(GET path RELATIVE_PART result)
  message("Relative part is \"${result}\"")

::

  Relative part is "a/b"
  Relative part is "c/d"
  Relative part is ""

パスを照会する例
""""""""""""""""

.. code-block:: cmake

  set(path "c:/a/b")
  cmake_path(GET path PARENT_PATH result)
  message("Parent path is \"${result}\"")

  set(path "c:/")
  cmake_path(GET path PARENT_PATH result)
  message("Parent path is \"${result}\"")

::

  Parent path is "c:/a"
  Parent path is "c:/"


.. _Path Query:

パスの要素を照会する
^^^^^^^^^^^^^^^^^^^^

``GET`` サブコマンドには、パスの中に特定の要素が存在しているかどうかを調べるための ``HAS_...`` で始まるサブコマンドがあります。
パスを構成する要素の意味については「`パスを構成する要素`_」の項を参照して下さい。

.. _HAS_ROOT_NAME:
.. _HAS_ROOT_DIRECTORY:
.. _HAS_ROOT_PATH:
.. _HAS_FILENAME:
.. _HAS_EXTENSION:
.. _HAS_STEM:
.. _HAS_RELATIVE_PART:
.. _HAS_PARENT_PATH:

.. code-block:: cmake

  cmake_path(HAS_ROOT_NAME <path-var> <out-var>)
  cmake_path(HAS_ROOT_DIRECTORY <path-var> <out-var>)
  cmake_path(HAS_ROOT_PATH <path-var> <out-var>)
  cmake_path(HAS_FILENAME <path-var> <out-var>)
  cmake_path(HAS_EXTENSION <path-var> <out-var>)
  cmake_path(HAS_STEM <path-var> <out-var>)
  cmake_path(HAS_RELATIVE_PART <path-var> <out-var>)
  cmake_path(HAS_PARENT_PATH <path-var> <out-var>)

上記は ``<path-var>`` の中に関連する要素が含まれていれば ``TRUE`` を、 それ以外は ``FALSE`` をそれぞれ ``<out-var>`` に格納します。
この時は、次の特殊なケースに注意して下さい：

* ``HAS_ROOT_PATH`` サブコマンドでは、``root-name`` または ``root-directory`` の少なくともどちらか一つが空ではない場合の結果は ``TRUE`` です。

* ``HAS_PARENT_PATH`` サブコマンドでは、「**CMake スタイルの root ディレクトリ**」は親ディレクトリを持ち、それは自分自身であるという概念が適用されます。そのため ``<path-var>`` が root ディレクトリの場合の結果は ``TRUE`` です。

.. _IS_ABSOLUTE:

.. code-block:: cmake

  cmake_path(IS_ABSOLUTE <path-var> <out-var>)


これは ``<path-var>`` に絶対パスを指定すると ``<out-var>`` は ``TRUE`` です。
CMake スタイルの絶対パスとは、追加で始点となるパスを参照することなく [#hint_for_absolete_path]_ 、ファイルの場所を明確に識別できるパスのことです。
ホストが Windows プラットフォームの場合、絶対パスを表現するには ``root-name`` と ``root-directory-separator``  の両方の要素が必要です。
ホストがそれ以外のプラットフォームでは ``root-directory-separator`` があれば十分です。
これは、 Windows の場合に ``IS_ABSOLUTE`` サブコマンドが ``FALSE`` になる一方で ``HAS_ROOT_DIRECTORY`` サブコマンドが ``TRUE`` になる可能性があるということに注意して下さい。


.. rubric:: 日本語訳注記

.. [#hint_for_absolete_path] 相対パスの場合は、まず始点のパスを認識し、そこから辿ってファイルの場所を識別するが、絶対パスの場合は始点は常に root ディレクトリである。

.. _IS_RELATIVE:

.. code-block:: cmake

  cmake_path(IS_RELATIVE <path-var> <out-var>)

これは ``<out-var>`` に ``IS_ABSOLUTE`` サブコマンドとは反対の結果を格納します。

.. _IS_PREFIX:

.. code-block:: cmake

  cmake_path(IS_PREFIX <path-var> <input> [NORMALIZE] <out-var>)

これは ``<path-var>`` が ``<input>`` の要素に含まれているかを確認します。

ここで ``NORMALIZE`` オプションを追加すると、確認する前に ``<path-var>`` と ``<input>`` は「:ref:`正規化 <Normalization>`」されます。

.. code-block:: cmake

  set(path "/a/b/c")
  cmake_path(IS_PREFIX path "/a/b/c/d" result) # result = true
  cmake_path(IS_PREFIX path "/a/b" result)     # result = false
  cmake_path(IS_PREFIX path "/x/y/z" result)   # result = false

  set(path "/a/b")
  cmake_path(IS_PREFIX path "/a/c/../b" NORMALIZE result)   # result = true

.. _Path COMPARE:
.. _COMPARE:

.. code-block:: cmake

  cmake_path(COMPARE <input1> EQUAL <input2> <out-var>)
  cmake_path(COMPARE <input1> NOT_EQUAL <input2> <out-var>)

これは文字列リテラルとして指定した二つのパスの字句表現（*lexical representations*）を比較します。
比較する前に、どちらのパスも正規化は行いませんが、複数の連続する ``directory-separator`` があれば一つまとめらます。
二つの字句が同じかどうかは、次の疑似コードによるアルゴリズム（疑似コード）に従って判断します：

::

  if(NOT <input1>.root_name() STREQUAL <input2>.root_name())
    return FALSE

  if(<input1>.has_root_directory() XOR <input2>.has_root_directory())
    return FALSE

  <input1> の相対部分（relative portion）が字句的に <input2> の相対部分
  と一致しない場合は  FALSE を返す。この比較はパスの要素ごとに実行される。
  そして <input1> と <input2> の全ての要素が同じであれば TRUE を返す。

.. note::
  他の大部分のサブコマンドとは異なり、この ``COMPARE`` サブコマンドは入力として変数名ではなく、文字列リテラルを受け取ります。


.. _Path Modification:

パスを編集する
^^^^^^^^^^^^^^

.. _cmake_path-SET:

.. code-block:: cmake

  cmake_path(SET <path-var> [NORMALIZE] <input>)

これは ``<input>`` をパスとして変数の ``<path-var>`` にセットします。
``<input>`` がホストにネィティブな書式の場合、スラッシュ（``/``）を使って CMake スタイルのパスに変換します。
ホストのプラットフォームが Windows の場合、長いファイル名のマーカー（``...``）も考慮されます。

``NORMALIZE`` オプションを指定すると、パスを変換した後に「:ref:`正規化 <Normalization>`」します。

たとえば：

.. code-block:: cmake

  set(native_path "c:\\a\\b/..\\c")
  cmake_path(SET path "${native_path}")
  message("CMake path is \"${path}\"")

  cmake_path(SET path NORMALIZE "${native_path}")
  message("Normalized CMake path is \"${path}\"")

の結果は::

  CMake path is "c:/a/b/../c"
  Normalized CMake path is "c:/a/c"

.. _APPEND:

.. code-block:: cmake

  cmake_path(APPEND <path-var> [<input>...] [OUTPUT_VARIABLE <out-var>])

これは、全ての ``<input>`` を ``<path-var>`` の最後に追加します。この時 ``directory-separator`` は ``/`` を使用します。
``<input>`` によっては ``<path-var>`` の以前の内容が破棄される場合があります。
``<input>`` はそれぞれ次のアルゴリズム（疑似コード） が適用されます：

::

  # <path> は <path-var> の中身

  if(<input>.is_absolute() OR
     (<input>.has_root_name() AND
      NOT <input>.root_name() STREQUAL <path>.root_name()))
    replace <path> with <input>
    return()
  endif()

  if(<input>.has_root_directory())
    remove any root-directory and the entire relative path from <path>
  elseif(<path>.has_filename() OR
         (NOT <path-var>.has_root_directory() OR <path>.is_absolute()))
    append directory-separator to <path>
  endif()

  append <input> omitting any root-name to <path>

.. _APPEND_STRING:

.. code-block:: cmake

  cmake_path(APPEND_STRING <path-var> [<input>...] [OUTPUT_VARIABLE <out-var>])

これは、全ての ``<input>`` を ``<path-var>`` の最後に追加します。ただし、この時 ``directory-separator`` は追加しません。

.. _REMOVE_FILENAME:

.. code-block:: cmake

  cmake_path(REMOVE_FILENAME <path-var> [OUTPUT_VARIABLE <out-var>])

これは、``<path-var>`` から :ref:`filename <FILENAME_DEF>` の要素（:ref:`GET ... FILENAME <GET_FILENAME>` サブコマンドが返す結果）を削除します。
その時、末尾にある ``directory-separator`` はそのまま残します。

``OUTPUT_VARIABLE`` オプションを指定しない場合、このコマンドから戻った後の `HAS_FILENAME`_ サブコマンドは ``<path-var>`` に対して ``FALSE`` を返します。

たとえば：

.. code-block:: cmake

  set(path "/a/b")
  cmake_path(REMOVE_FILENAME path)
  message("First path is \"${path}\"")

  # ここで「ファイル名」に相当する要素は空なので、次のコマンドは何もしない
  cmake_path(REMOVE_FILENAME path)
  message("Second path is \"${result}\"")

この結果は::

  First path is "/a/"
  Second path is "/a/"

.. _REPLACE_FILENAME:

.. code-block:: cmake

  cmake_path(REPLACE_FILENAME <path-var> <input> [OUTPUT_VARIABLE <out-var>])

これは、``<path-var>`` にある :ref:`filename <FILENAME_DEF>` の要素を ``<input>`` で置き換えます。
``<path-var>`` に ``filename`` の要素がない場合（すなわち `HAS_FILENAME`_ サブコマンドの返り値が ``FALSE`` の場合）、パスは何も変更されません。
このサブコマンドは次の操作と等価です：

.. code-block:: cmake

  cmake_path(HAS_FILENAME path has_filename)
  if(has_filename)
    cmake_path(REMOVE_FILENAME path)
    cmake_path(APPEND path input);
  endif()

.. _REMOVE_EXTENSION:

.. code-block:: cmake

  cmake_path(REMOVE_EXTENSION <path-var> [LAST_ONLY]
                                         [OUTPUT_VARIABLE <out-var>])

これは、``<path-var>`` の中にある :ref:`拡張子 <EXTENSION_DEF>` を削除します（存在しなければ、パスは何も変更されません）。

.. _REPLACE_EXTENSION:

.. code-block:: cmake

  cmake_path(REPLACE_EXTENSION <path-var> [LAST_ONLY] <input>
                               [OUTPUT_VARIABLE <out-var>])

これは、:ref:`拡張子 <EXTENSION_DEF>` を ``<input>`` で置き換えます。
このサブコマンドは次の操作と等価です：

.. code-block:: cmake

  cmake_path(REMOVE_EXTENSION path)
  if(NOT "input" MATCHES "^\\.")
    cmake_path(APPEND_STRING path ".")
  endif()
  cmake_path(APPEND_STRING path "input")


.. _Path Generation:

パスを変換する
^^^^^^^^^^^^^^

.. _NORMAL_PATH:

.. code-block:: cmake

  cmake_path(NORMAL_PATH <path-var> [OUTPUT_VARIABLE <out-var>])

これは、「:ref:`パスを正規化する <Normalization>`」で説明した手順に従って ``<path-var>`` を正規化します。

.. _cmake_path-RELATIVE_PATH:
.. _RELATIVE_PATH:

.. code-block:: cmake

  cmake_path(RELATIVE_PATH <path-var> [BASE_DIRECTORY <input>]
                                      [OUTPUT_VARIABLE <out-var>])

これは、``<path-var>`` を ``BASE_DIRECTORY`` のオプションとして渡した ``<input>`` をベース・ディレクトリとした相対パスに変更します。
``BASE_DIRECTORY`` オプションを指定しない場合は、デフォルトのベース・ディレクトリとして CMake 変数の :variable:`CMAKE_CURRENT_SOURCE_DIR` を使います。

なお、ここで相対パスを計算する際に使用するアルゴリズムは、C++ の `std::filesystem::path::lexically_relative <https://en.cppreference.com/w/cpp/filesystem/path/lexically_normal>`_ で使っているものと同じです。

.. _ABSOLUTE_PATH:

.. code-block:: cmake

  cmake_path(ABSOLUTE_PATH <path-var> [BASE_DIRECTORY <input>] [NORMALIZE]
                                      [OUTPUT_VARIABLE <out-var>])

これは、``<path-var>`` が相対パス（`IS_RELATIVE`_ サブコマンドの返り値が ``TRUE`` のパス ）の場合に ``BASE_DIRECTORY`` オプションで指定した ``<input>`` をベース・ディレクトリとして評価します。
``BASE_DIRECTORY`` オプションを指定しない場合は、デフォルトのベース・ディレクトリとして CMake 変数の :variable:`CMAKE_CURRENT_SOURCE_DIR` を使います。

ここで ``NORMALIZE`` オプションを追加すると、評価した後にパスは :ref:`正規化 <Normalization>` されます。

この ``cmake_path()`` コマンドは実際のファイルシステムにアクセスしないので、シンボリックリンクは解決されず、パスの先頭にあるチルダ文字（"``~``"）はシェルのように展開されません。
シンボリックリンクを解決し、チルダ文字が正しく展開された実際のパスに変換するには、代わりに :command:`file(REAL_PATH)` コマンドを使って下さい。

ネィティブ・スタイルに変換する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

このセクションにあるコマンドで、クロス・コンパイル時の「ネィティブ」とはターゲット・プラットフォームではなく、ホストのプラットフォームを指します。

.. _cmake_path-NATIVE_PATH:
.. _NATIVE_PATH:

.. code-block:: cmake

  cmake_path(NATIVE_PATH <path-var> [NORMALIZE] <out-var>)

これは、CMake スタイルの ``<path-var>`` をプラットフォーム固有のスラッシュ（Windows の場合は "``\``"、それ以外は "``/``"）を含む「ネィティブ」なパスに変換します。

ここで ``NORMALIZE`` オプションを追加すると、変換する前にパスは :ref:`正規化 <Normalization>` されます。

.. _CONVERT:
.. _cmake_path-TO_CMAKE_PATH_LIST:
.. _TO_CMAKE_PATH_LIST:

.. code-block:: cmake

  cmake_path(CONVERT <input> TO_CMAKE_PATH_LIST <out-var> [NORMALIZE])

これは、ホストにネィティブなパスである ``<input>`` を、スラッシュ（"``/``"） を使った CMake スタイルのパスに変換します。
ホストのプラットフォームが Windows の場合、長いファイル名のマーカー（``...``）も考慮されます。
``<input>`` には単一のパス、または ``$ENV{PATH}`` のような環境変数の検索パスを渡すことができます。
この検索パスはセミコロン文字（"``;``"）で区切られた CMake スタイルのパスを要素とするリストに変換されます（すなわち、Windows 以外のプラットフォームの場合は、コロン文字 "``:``" がセミコロン文字 "``;``" で置き換えられます）。
変換した結果は ``<out-var>`` に格納されます。

ここで ``NORMALIZE`` オプションを追加すると、変換する前にパスは :ref:`正規化 <Normalization>` されます。

.. note::
  他の大部分のサブコマンドとは異なり、この ``CONVERT`` サブコマンドは入力として変数名ではなく、文字列リテラルを受け取ります。

.. _cmake_path-TO_NATIVE_PATH_LIST:
.. _TO_NATIVE_PATH_LIST:

.. code-block:: cmake

  cmake_path(CONVERT <input> TO_NATIVE_PATH_LIST <out-var> [NORMALIZE])

これは、CMake スタイルのパスである ``<input>`` をプラットフォーム固有のスラッシュ（Windows の場合は "``\``"、それ以外は "``/``"）を含む「ネィティブ」なパスに変換します。
``<input>`` には単一のパス、または CMake スタイルのパスを要素とするリストを渡すことができます。
このリストはネィティブの検索パス形式（パスは、Windows プラットフォームではセミコロン "``;``"、それ以外のプラットフォームではコロン "``:``" で区切られます）に変換されます。
変換した結果は ``<out-var>`` に格納されます。

ここで ``NORMALIZE`` オプションを追加すると、変換する前にパスは :ref:`正規化 <Normalization>` されます。

.. note::
  他の大部分のサブコマンドとは異なり、この ``CONVERT`` サブコマンドは入力として変数名ではなく、文字列リテラルを受け取ります。

たとえば：

.. code-block:: cmake

  set(paths "/a/b/c" "/x/y/z")
  cmake_path(CONVERT "${paths}" TO_NATIVE_PATH_LIST native_paths)
  message("Native path list is \"${native_paths}\"")

この結果は Windows プラットフォームでは::

  Native path list is "\a\b\c;\x\y\z"

それ以外のプラットフォームでは::

  Native path list is "/a/b/c:/x/y/z"

ハッシュ値
^^^^^^^^^^

.. _HASH:

.. code-block:: cmake

    cmake_path(HASH <path-var> <out-var>)

``<path-var>`` のハッシュ値を計算します。
これは、たとえば二つのパスの ``p1`` と ``p2`` を :ref:`COMPARE ... EQUAL <COMPARE>` サブコマンドで比較するように、``p1`` のハッシュ値が ``p2`` のハッシュ値と同じになるように計算します。
ハッシュ値を計算する前にパスは必ず :ref:`正規化 <Normalization>` されます。
