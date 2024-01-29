set_property
------------

指定したスコープ内で有効な名前付きのプロパティをオブジェクトにセットする。

.. code-block:: cmake

  set_property(<GLOBAL                      |
                DIRECTORY [<dir>]           |
                TARGET    [<target1> ...]   |
                SOURCE    [<src1> ...]
                          [DIRECTORY <dirs> ...]
                          [TARGET_DIRECTORY <targets> ...] |
                INSTALL   [<file1> ...]     |
                TEST      [<test1> ...]
                          [DIRECTORY <dir>] |
                CACHE     [<entry1> ...]    >
               [APPEND] [APPEND_STRING]
               PROPERTY <name> [<value1> ...])

スコープにある0個以上のオブジェクトに1個のプロパティをセットします。

最初の引数は、プロパティをセットするスコープを表します。
次のいずれかを指定して下さい：

``GLOBAL``
  スコープは一つだけで重複させることはできない。
  名前は付与できない。

``DIRECTORY``
  デフォルトは現在の作業ディレクトリ。
  CMake が認識している他のディレクトリ（絶対パスまたは相対パス）もスコープに指定できる。
  なお相対パスは :variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとしてパスを計算する。
  :command:`set_directory_properties` コマンドも参照のこと。

  .. versionadded:: 3.19
    ``<dir>`` に :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できるようになった。

``TARGET``
  0個以上のビルド・ターゲットをスコープに指定できる。
  :command:`set_target_properties` コマンドも参照のこと。

  ただし :ref:`Alias Targets` にプロパティはセットできない。

``SOURCE``
  0個以上の既存のソース・ファイルをスコープに指定できる。
  このコマンドを呼び出した ``CMakeLists.txt`` と同じディレクトリに追加したソース・ファイルが対象である。

  .. versionadded:: 3.18
    次のサブ・オプションの一つ、または両方を使って、他のディレクトリ・スコープで Visibility をセットできるようになった。

    ``DIRECTORY <dirs>...``
      指定した ``<dir>`` ディレクトリごとのスコープでソース・ファイルに対するプロパティがセットされる。
      ``<dir>`` は :command:`add_subdirectory` コマンドの呼び出しを通してディレクトリを追加するか、:variable:`CMAKE_CURRENT_SOURCE_DIR` であることにより、CMake が既に認識しているディレクトリを指定して下さい。
      なお相対パスは :variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとしてパスを計算する。

      .. versionadded:: 3.19
        ``<dir>`` に :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できるようになった。

    ``TARGET_DIRECTORY <targets>...``
      The source file property will be set in each of the directory scopes where any of the specified ``<targets>`` were created (the ``<targets>`` must therefore already exist).

  See also the :command:`set_source_files_properties` command.

``INSTALL``
  .. versionadded:: 3.1

  Scope may name zero or more installed file paths.
  These are made available to CPack to influence deployment.

  Both the property key and value may use generator expressions.
  Specific properties may apply to installed files and/or directories.

  Path components have to be separated by forward slashes, must be normalized and are case sensitive.

  To reference the installation prefix itself with a relative path use ``.``.

  Currently installed file properties are only defined for the WIX generator where the given paths are relative to the installation prefix.

``TEST``
  Scope is limited to the directory the command is called in.
  It may name zero or more existing tests. See also command :command:`set_tests_properties`.

  Test property values may be specified using :manual:`generator expressions <cmake-generator-expressions(7)>` for tests created by the :command:`add_test(NAME)` signature.

  .. versionadded:: 3.28

    Visibility can be set in other directory scopes using the following sub-option:

    ``DIRECTORY <dir>``
      The test property will be set in the ``<dir>`` directory's scope.
      CMake must already know about this directory, either by having added it through a call to :command:`add_subdirectory` or it being the top level source directory.
      Relative paths are treated as relative to the current source directory.
      ``<dir>`` には :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できる。

``CACHE``
  Scope must name zero or more existing cache entries.

The required ``PROPERTY`` option is immediately followed by the name of the property to set.
Remaining arguments are used to compose the property value in the form of a semicolon-separated list.

If the ``APPEND`` option is given the list is appended to any existing property value (except that empty values are ignored and not appended).
If the ``APPEND_STRING`` option is given the string is appended to any existing property value as string, i.e. it results in a longer string and not a list of strings.
When using ``APPEND`` or ``APPEND_STRING`` with a property defined to support ``INHERITED`` behavior (see :command:`define_property`), no inheriting occurs when finding the initial value to append to.
If the property is not already directly set in the nominated scope, the command will behave as though ``APPEND`` or ``APPEND_STRING`` had not been given.

.. note::

  The :prop_sf:`GENERATED` source file property may be globally visible.
  See its documentation for details.

参考情報
^^^^^^^^

* :command:`define_property`
* :command:`get_property`
* The :manual:`cmake-properties(7)` manual for a list of properties
  in each scope.
