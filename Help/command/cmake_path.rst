cmake_path
----------

.. versionadded:: 3.20

**CMake 上のパス** を操作するコマンド。
CMake 上のパスを構成している要素だけ処理し、実際のファイルシステムとのやりとりは行なわない。
そのため CMake 上のパスは「実際には存在しないパス」とか「現在のファイルシステムやプットフォームに存在することが許されないパス」を表す場合がある。
これに対し、ファイルシステムとやり取りする操作については :command:`file` コマンドを参照のこと。

.. note::

  ``cmake_path`` コマンドはターゲットではなく、ビルドシステム（すなわちホストのプラットフォーム）の形式でパスを処理します。
  クロス・コンパイルで、パスにホストのプラットフォームでは扱えない要素（たとえばホストが Windows でない時のドライブ名など）が含まれていたら、その結果は不明です。

概要
^^^^

.. parsed-literal::

  `規則`_

  `パスを構成する要素`_

  `パス変数を作成する`_

  `パスの正規化`_

  `パスの分解`_
    cmake_path(`GET`_ <path-var> :ref:`ROOT_NAME <GET_ROOT_NAME>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`ROOT_DIRECTORY <GET_ROOT_DIRECTORY>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`ROOT_PATH <GET_ROOT_PATH>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`FILENAME <GET_FILENAME>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`EXTENSION <GET_EXTENSION>` [LAST_ONLY] <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`STEM <GET_STEM>` [LAST_ONLY] <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`RELATIVE_PART <GET_RELATIVE_PART>` <out-var>)
    cmake_path(`GET`_ <path-var> :ref:`PARENT_PATH <GET_PARENT_PATH>` <out-var>)

  `Query`_
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

  `Modification`_
    cmake_path(:ref:`SET <cmake_path-SET>` <path-var> [NORMALIZE] <input>)
    cmake_path(`APPEND`_ <path-var> [<input>...] [OUTPUT_VARIABLE <out-var>])
    cmake_path(`APPEND_STRING`_ <path-var> [<input>...] [OUTPUT_VARIABLE <out-var>])
    cmake_path(`REMOVE_FILENAME`_ <path-var> [OUTPUT_VARIABLE <out-var>])
    cmake_path(`REPLACE_FILENAME`_ <path-var> <input> [OUTPUT_VARIABLE <out-var>])
    cmake_path(`REMOVE_EXTENSION`_ <path-var> [LAST_ONLY] [OUTPUT_VARIABLE <out-var>])
    cmake_path(`REPLACE_EXTENSION`_ <path-var> [LAST_ONLY] <input> [OUTPUT_VARIABLE <out-var>])

  `Generation`_
    cmake_path(`NORMAL_PATH`_ <path-var> [OUTPUT_VARIABLE <out-var>])
    cmake_path(`RELATIVE_PATH`_ <path-var> [BASE_DIRECTORY <input>] [OUTPUT_VARIABLE <out-var>])
    cmake_path(`ABSOLUTE_PATH`_ <path-var> [BASE_DIRECTORY <input>] [NORMALIZE] [OUTPUT_VARIABLE <out-var>])

  `Native Conversion`_
    cmake_path(`NATIVE_PATH`_ <path-var> [NORMALIZE] <out-var>)
    cmake_path(`CONVERT`_ <input> `TO_CMAKE_PATH_LIST`_ <out-var> [NORMALIZE])
    cmake_path(`CONVERT`_ <input> `TO_NATIVE_PATH_LIST`_ <out-var> [NORMALIZE])

  `Hashing`_
    cmake_path(`HASH`_ <path-var> <out-var>)

.. _Conversions:

規則
^^^^

このコマンドのドキュメントでは次の規則に準じます：

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

CMake 上のパスは次のような構造を持ちます（全ての要素はオプションですが、いくつかの制限があります）：

::

  root-name root-directory-separator (item-name directory-separator)* filename

``root-name``
  複数の root ディレクトリ（たとえば ``"C:"`` や ``"//myserver"`` など）があるファイルシステム上での root を表す（オプション）。

``root-directory-separator``
  ディレクトリの区切り文字。
  この文字が含まれているパスは絶対パスであることを示す。
  この区切り文字が無く、先頭の要素が ``root-name`` 以外の ``item-name`` の場合、そのパスは相対パスになる。

``item-name``
  ディレクトリの区切り文字ではない文字列リテラル。
  この名前は、一個のファイル、一個のハード・リンク、一個のシンボリック・リンク、あるいは一個のディレクトリを表す。
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
  実のところ ``filename`` はパスの最後にある ``item-name`` なので、一個のハードリンク、一個のシンボリック・リンク、または一個のディレクトリにすることも可能である。

  ``filename`` には一個の *拡張子* をつけることが可能である。
  デフォルトで、拡張子はピリオドから ``filename`` の末尾までの部分文字列として定義される。
  ``LAST_ONLY`` というキーワードを受け取るコマンドでは、``LAST_ONLY`` が拡張子でピリオドを除く部分文字列に置き換わる。

  拡張子には、次に示す例外が適用される：

    * もし ``filename`` の先頭の文字が一個のドット文字（"``.``"）だったら、拡張子の検出では無視される（たとえば ``".profile"`` は拡張子なしとして扱う）

    * もし ``filename`` が ``.`` または ``..`` だったら、拡張子なしとして扱う

  *STEM* とは ``filename`` の拡張子よりも前の部分文字列を指す。

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

`Modification`_ と `Generation`_ のサブコマンドは、実行結果をその場で保存することも、あるいは ``OUTPUT_VARIABLE`` というキーワードを持つ別の変数に保存することもできます。 
それ以外のサブコマンドは全て ``<out-var>`` 変数に結果を保存します。

.. _Normalization:

パスの正規化
^^^^^^^^^^^^

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

パスの分解
^^^^^^^^^^

.. _GET:
.. _GET_ROOT_NAME:
.. _GET_ROOT_DIRECTORY:
.. _GET_ROOT_PATH:
.. _GET_FILENAME:
.. _GET_EXTENSION:
.. _GET_STEM:
.. _GET_RELATIVE_PART:
.. _GET_PARENT_PATH:

The following forms of the ``GET`` subcommand each retrieve a different
component or group of components from a path.  See
`Path Structure And Terminology`_ for the meaning of each path component.

.. code-block:: cmake

  cmake_path(GET <path-var> ROOT_NAME <out-var>)
  cmake_path(GET <path-var> ROOT_DIRECTORY <out-var>)
  cmake_path(GET <path-var> ROOT_PATH <out-var>)
  cmake_path(GET <path-var> FILENAME <out-var>)
  cmake_path(GET <path-var> EXTENSION [LAST_ONLY] <out-var>)
  cmake_path(GET <path-var> STEM [LAST_ONLY] <out-var>)
  cmake_path(GET <path-var> RELATIVE_PART <out-var>)
  cmake_path(GET <path-var> PARENT_PATH <out-var>)

If a requested component is not present in the path, an empty string will be
stored in ``<out-var>``.  For example, only Windows systems have the concept
of a ``root-name``, so when the host machine is non-Windows, the ``ROOT_NAME``
subcommand will always return an empty string.

For ``PARENT_PATH``, if the `HAS_RELATIVE_PART`_ subcommand returns false,
the result is a copy of ``<path-var>``.  Note that this implies that a root
directory is considered to have a parent, with that parent being itself.
Where `HAS_RELATIVE_PART`_ returns true, the result will essentially be
``<path-var>`` with one less element.

Root examples
"""""""""""""

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

Filename examples
"""""""""""""""""

.. code-block:: cmake

  set(path "/a/b")
  cmake_path(GET path FILENAME filename)
  message("First filename is \"${filename}\"")

  # Trailing slash means filename is empty
  set(path "/a/b/")
  cmake_path(GET path FILENAME filename)
  message("Second filename is \"${filename}\"")

::

  First filename is "b"
  Second filename is ""

Extension and stem examples
"""""""""""""""""""""""""""

.. code-block:: cmake

  set(path "name.ext1.ext2")

  cmake_path(GET path EXTENSION fullExt)
  cmake_path(GET path STEM fullStem)
  message("Full extension is \"${fullExt}\"")
  message("Full stem is \"${fullStem}\"")

  # Effect of LAST_ONLY
  cmake_path(GET path EXTENSION LAST_ONLY lastExt)
  cmake_path(GET path STEM LAST_ONLY lastStem)
  message("Last extension is \"${lastExt}\"")
  message("Last stem is \"${lastStem}\"")

  # Special cases
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

Relative part examples
""""""""""""""""""""""

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

Path traversal examples
"""""""""""""""""""""""

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

Query
^^^^^

Each of the ``GET`` subcommands has a corresponding ``HAS_...``
subcommand which can be used to discover whether a particular path
component is present.  See `Path Structure And Terminology`_ for the
meaning of each path component.

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

Each of the above follows the predictable pattern of setting ``<out-var>``
to true if the path has the associated component, or false otherwise.
Note the following special cases:

* For ``HAS_ROOT_PATH``, a true result will only be returned if at least one
  of ``root-name`` or ``root-directory`` is non-empty.

* For ``HAS_PARENT_PATH``, the root directory is also considered to have a
  parent, which will be itself.  The result is true except if the path
  consists of just a :ref:`filename <FILENAME_DEF>`.

.. _IS_ABSOLUTE:

.. code-block:: cmake

  cmake_path(IS_ABSOLUTE <path-var> <out-var>)

Sets ``<out-var>`` to true if ``<path-var>`` is absolute.  An absolute path
is a path that unambiguously identifies the location of a file without
reference to an additional starting location.  On Windows, this means the
path must have both a ``root-name`` and a ``root-directory-separator`` to be
considered absolute.  On other platforms, just a ``root-directory-separator``
is sufficient.  Note that this means on Windows, ``IS_ABSOLUTE`` can be
false while ``HAS_ROOT_DIRECTORY`` can be true.

.. _IS_RELATIVE:

.. code-block:: cmake

  cmake_path(IS_RELATIVE <path-var> <out-var>)

This will store the opposite of ``IS_ABSOLUTE`` in ``<out-var>``.

.. _IS_PREFIX:

.. code-block:: cmake

  cmake_path(IS_PREFIX <path-var> <input> [NORMALIZE] <out-var>)

Checks if ``<path-var>`` is the prefix of ``<input>``.

When the ``NORMALIZE`` option is specified, ``<path-var>`` and ``<input>``
are :ref:`normalized <Normalization>` before the check.

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

Compares the lexical representations of two paths provided as string literals.
No normalization is performed on either path, except multiple consecutive
directory separators are effectively collapsed into a single separator.
Equality is determined according to the following pseudo-code logic:

::

  if(NOT <input1>.root_name() STREQUAL <input2>.root_name())
    return FALSE

  if(<input1>.has_root_directory() XOR <input2>.has_root_directory())
    return FALSE

  Return FALSE if a relative portion of <input1> is not lexicographically
  equal to the relative portion of <input2>. This comparison is performed path
  component-wise. If all of the components compare equal, then return TRUE.

.. note::
  Unlike most other ``cmake_path()`` subcommands, the ``COMPARE`` subcommand
  takes literal strings as input, not the names of variables.


.. _Path Modification:

Modification
^^^^^^^^^^^^

.. _cmake_path-SET:

.. code-block:: cmake

  cmake_path(SET <path-var> [NORMALIZE] <input>)

Assign the ``<input>`` path to ``<path-var>``.  If ``<input>`` is a native
path, it is converted into a cmake-style path with forward-slashes
(``/``). On Windows, the long filename marker is taken into account.

When the ``NORMALIZE`` option is specified, the path is :ref:`normalized
<Normalization>` after the conversion.

For example:

.. code-block:: cmake

  set(native_path "c:\\a\\b/..\\c")
  cmake_path(SET path "${native_path}")
  message("CMake path is \"${path}\"")

  cmake_path(SET path NORMALIZE "${native_path}")
  message("Normalized CMake path is \"${path}\"")

Output::

  CMake path is "c:/a/b/../c"
  Normalized CMake path is "c:/a/c"

.. _APPEND:

.. code-block:: cmake

  cmake_path(APPEND <path-var> [<input>...] [OUTPUT_VARIABLE <out-var>])

Append all the ``<input>`` arguments to the ``<path-var>`` using ``/`` as
the ``directory-separator``.  Depending on the ``<input>``, the previous
contents of ``<path-var>`` may be discarded.  For each ``<input>`` argument,
the following algorithm (pseudo-code) applies:

::

  # <path> is the contents of <path-var>

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

Append all the ``<input>`` arguments to the ``<path-var>`` without adding any
``directory-separator``.

.. _REMOVE_FILENAME:

.. code-block:: cmake

  cmake_path(REMOVE_FILENAME <path-var> [OUTPUT_VARIABLE <out-var>])

Removes the :ref:`filename <FILENAME_DEF>` component (as returned by
:ref:`GET ... FILENAME <GET_FILENAME>`) from ``<path-var>``.  After removal,
any trailing ``directory-separator`` is left alone, if present.

If ``OUTPUT_VARIABLE`` is not given, then after this function returns,
`HAS_FILENAME`_ returns false for ``<path-var>``.

For example:

.. code-block:: cmake

  set(path "/a/b")
  cmake_path(REMOVE_FILENAME path)
  message("First path is \"${path}\"")

  # filename is now already empty, the following removes nothing
  cmake_path(REMOVE_FILENAME path)
  message("Second path is \"${result}\"")

Output::

  First path is "/a/"
  Second path is "/a/"

.. _REPLACE_FILENAME:

.. code-block:: cmake

  cmake_path(REPLACE_FILENAME <path-var> <input> [OUTPUT_VARIABLE <out-var>])

Replaces the :ref:`filename <FILENAME_DEF>` component from ``<path-var>``
with ``<input>``.  If ``<path-var>`` has no filename component (i.e.
`HAS_FILENAME`_ returns false), the path is unchanged.  The operation is
equivalent to the following:

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

Removes the :ref:`extension <EXTENSION_DEF>`, if any, from ``<path-var>``.

.. _REPLACE_EXTENSION:

.. code-block:: cmake

  cmake_path(REPLACE_EXTENSION <path-var> [LAST_ONLY] <input>
                               [OUTPUT_VARIABLE <out-var>])

Replaces the :ref:`extension <EXTENSION_DEF>` with ``<input>``.  Its effect
is equivalent to the following:

.. code-block:: cmake

  cmake_path(REMOVE_EXTENSION path)
  if(NOT "input" MATCHES "^\\.")
    cmake_path(APPEND_STRING path ".")
  endif()
  cmake_path(APPEND_STRING path "input")


.. _Path Generation:

Generation
^^^^^^^^^^

.. _NORMAL_PATH:

.. code-block:: cmake

  cmake_path(NORMAL_PATH <path-var> [OUTPUT_VARIABLE <out-var>])

Normalize ``<path-var>`` according the steps described in :ref:`Normalization`.

.. _cmake_path-RELATIVE_PATH:
.. _RELATIVE_PATH:

.. code-block:: cmake

  cmake_path(RELATIVE_PATH <path-var> [BASE_DIRECTORY <input>]
                                      [OUTPUT_VARIABLE <out-var>])

Modifies ``<path-var>`` to make it relative to the ``BASE_DIRECTORY`` argument.
If ``BASE_DIRECTORY`` is not specified, the default base directory will be
:variable:`CMAKE_CURRENT_SOURCE_DIR`.

For reference, the algorithm used to compute the relative path is the same
as that used by C++
`std::filesystem::path::lexically_relative
<https://en.cppreference.com/w/cpp/filesystem/path/lexically_normal>`_.

.. _ABSOLUTE_PATH:

.. code-block:: cmake

  cmake_path(ABSOLUTE_PATH <path-var> [BASE_DIRECTORY <input>] [NORMALIZE]
                                      [OUTPUT_VARIABLE <out-var>])

If ``<path-var>`` is a relative path (`IS_RELATIVE`_ is true), it is evaluated
relative to the given base directory specified by ``BASE_DIRECTORY`` option.
If ``BASE_DIRECTORY`` is not specified, the default base directory will be
:variable:`CMAKE_CURRENT_SOURCE_DIR`.

When the ``NORMALIZE`` option is specified, the path is :ref:`normalized
<Normalization>` after the path computation.

Because ``cmake_path()`` does not access the filesystem, symbolic links are
not resolved and any leading tilde is not expanded.  To compute a real path
with symbolic links resolved and leading tildes expanded, use the
:command:`file(REAL_PATH)` command instead.

Native Conversion
^^^^^^^^^^^^^^^^^

For commands in this section, *native* refers to the host platform, not the
target platform when cross-compiling.

.. _cmake_path-NATIVE_PATH:
.. _NATIVE_PATH:

.. code-block:: cmake

  cmake_path(NATIVE_PATH <path-var> [NORMALIZE] <out-var>)

Converts a cmake-style ``<path-var>`` into a native path with
platform-specific slashes (``\`` on Windows hosts and ``/`` elsewhere).

When the ``NORMALIZE`` option is specified, the path is :ref:`normalized
<Normalization>` before the conversion.

.. _CONVERT:
.. _cmake_path-TO_CMAKE_PATH_LIST:
.. _TO_CMAKE_PATH_LIST:

.. code-block:: cmake

  cmake_path(CONVERT <input> TO_CMAKE_PATH_LIST <out-var> [NORMALIZE])

Converts a native ``<input>`` path into a cmake-style path with forward
slashes (``/``).  On Windows hosts, the long filename marker is taken into
account.  The input can be a single path or a system search path like
``$ENV{PATH}``.  A search path will be converted to a cmake-style list
separated by ``;`` characters (on non-Windows platforms, this essentially
means ``:`` separators are replaced with ``;``).  The result of the
conversion is stored in the ``<out-var>`` variable.

When the ``NORMALIZE`` option is specified, the path is :ref:`normalized
<Normalization>` before the conversion.

.. note::
  Unlike most other ``cmake_path()`` subcommands, the ``CONVERT`` subcommand
  takes a literal string as input, not the name of a variable.

.. _cmake_path-TO_NATIVE_PATH_LIST:
.. _TO_NATIVE_PATH_LIST:

.. code-block:: cmake

  cmake_path(CONVERT <input> TO_NATIVE_PATH_LIST <out-var> [NORMALIZE])

Converts a cmake-style ``<input>`` path into a native path with
platform-specific slashes (``\`` on Windows hosts and ``/`` elsewhere).
The input can be a single path or a cmake-style list.  A list will be
converted into a native search path (``;``-separated on Windows,
``:``-separated on other platforms).  The result of the conversion is
stored in the ``<out-var>`` variable.

When the ``NORMALIZE`` option is specified, the path is :ref:`normalized
<Normalization>` before the conversion.

.. note::
  Unlike most other ``cmake_path()`` subcommands, the ``CONVERT`` subcommand
  takes a literal string as input, not the name of a variable.

For example:

.. code-block:: cmake

  set(paths "/a/b/c" "/x/y/z")
  cmake_path(CONVERT "${paths}" TO_NATIVE_PATH_LIST native_paths)
  message("Native path list is \"${native_paths}\"")

Output on Windows::

  Native path list is "\a\b\c;\x\y\z"

Output on all other platforms::

  Native path list is "/a/b/c:/x/y/z"

Hashing
^^^^^^^

.. _HASH:

.. code-block:: cmake

    cmake_path(HASH <path-var> <out-var>)

Compute a hash value of ``<path-var>`` such that for two paths ``p1`` and
``p2`` that compare equal (:ref:`COMPARE ... EQUAL <COMPARE>`), the hash
value of ``p1`` is equal to the hash value of ``p2``.  The path is always
:ref:`normalized <Normalization>` before the hash is computed.
