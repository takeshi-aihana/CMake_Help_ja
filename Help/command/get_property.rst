get_property
------------

プロパティを取得する。

.. code-block:: cmake

  get_property(<variable>
               <GLOBAL             |
                DIRECTORY [<dir>]  |
                TARGET    <target> |
                SOURCE    <source>
                          [DIRECTORY <dir> | TARGET_DIRECTORY <target>] |
                INSTALL   <file>   |
                TEST      <test>
                          [DIRECTORY <dir>] |
                CACHE     <entry>  |
                VARIABLE           >
               PROPERTY <name>
               [SET | DEFINED | BRIEF_DOCS | FULL_DOCS])

ディレクトリなど任意のスコープにある任意のオブジェクトから、名前の付いた属性値（**プロパティ**）を一つ取得します。

このコマンドの最初の引数は、取得した結果を格納する変数です。
二番目の引数には、プロパティを取得するスコープを指定します。
スコープは次のいずれかを指定して下さい：

``GLOBAL``
  このスコープは一意であり、プロパティの名前は指定できない。

``DIRECTORY``
  スコープのデフォルトは現在のディレクトリであるが、別のディレクトリ（CMake が既に読み込んだ絶対パスまたは相対パス）を ``<dir>`` に指定できる。
  相対パスの場合は :variable:`CMAKE_CURRENT_SOURCE_DIR` を起点したディレクトリとして扱う。
  :command:`get_directory_property` コマンドも参照のこと。

  .. versionadded:: 3.19
    ``<dir>`` に :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できるようになった。

``TARGET``
  スコープとして ``<target>`` に既存のターゲットを一つ指定する。
  :command:`get_target_property` コマンドも参照のこと。

``SOURCE``
  スコープとして ``<source>`` に一個のソースファイルを指定する。
  デフォルトで、:variable:`CMAKE_CURRENT_SOURCE_DIR` にあるソース・ファイルからプロパティを取得する。

  .. versionadded:: 3.18
    ``DIRECTORY`` スコープに、次のサブオプションが追加された：

    ``DIRECTORY <dir>``
      ``SOURCE`` スコープで取得するプロパティは ``<dir>`` に指定したディレクトリのソース・ファイルから取得する。
      ただし ``<dir>`` に指定できるのは、:command:`add_subdirectory` コマンドやプロジェクト最上位のディレクトリとして、既に CMake が認識しているディレクトリにすること。
      相対パスは :variable:`CMAKE_CURRENT_SOURCE_DIR` を起点したディレクトリとして扱う。

      .. versionadded:: 3.19
        ``<dir>`` に :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できるようになった。

    ``TARGET_DIRECTORY <target>``
      ``SOURCE`` スコープのプロパティを ``<target>`` にあるディレクトリのソース・ファイルから読み込む（この ``<target>`` ディレクトリは既に存在しているディレクトリを指定すること）。

  :command:`get_source_file_property` コマンドも参照のこと。

``INSTALL``
  .. versionadded:: 3.1

  スコープとして ``file`` にインストール済みのファイルのパスを一つ指定する。

``TEST``
  スコープとして ``<test>`` には既存のテストを一つ指定する。
  :command:`get_test_property` コマンドも参照のこと。

  .. versionadded:: 3.28
    ``DIRECTORY`` スコープに、次のサブオプションが追加された：

    ``DIRECTORY <dir>``
      ``TEST`` スコープで取得するプロパティは ``<dir>`` に指定したディレクトリのテストから取得する。
      ただし ``<dir>`` に指定できるのは、 :command:`add_subdirectory` コマンドやプロジェクト最上位のディレクトリとして、既に CMake が認識しているディレクトリにすること。
      相対パスは 
      相対パスは :variable:`CMAKE_CURRENT_SOURCE_DIR` を起点したディレクトリとして扱う。

      .. versionadded:: 3.19
        ``<dir>`` に :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できるようになった。

``CACHE``
  スコープとして ``<entry>`` には既存のキャッシュ変数を一つ指定する。

``VARIABLE``
  このスコープは一意であり、プロパティの名前は指定できない。

必須の ``PROPERTY`` オプションに続く ``<name>`` には、取得するプロパティの名前を指定します。
プロパティが存在しない場合は空の値が返されますが、明示的に空の値がセットされているようなプロパティの場合は親にあたるスコープから値を継承できます（:command:`define_property` コマンドを参照のこと）。

``SET`` オプションを指定すると、``<variable>``  には、そのプロパティがセットされているかどうかを示す論理値が返されます。
``DEFINED`` オプションを指定すると、``<variable>`` には、:command:`define_property` コマンドで定義されたプロパティかどうかを示す論理値が返されます。

``BRIEF_DOCS`` または ``FULL_DOCS`` プロパティを指定すると、``<variable>`` はそのプロパティのドキュメントを含む文字列が返されます。
この時に、定義されていないプロパティのドキュメントを要求すると ``<variable>`` には ``NOTFOUND`` が返されます。

.. note::

  :prop_sf:`GENERATED` なソース・ファイルのプロパティが ``GLOBAL`` スコープに表示される場合があります。
  詳細は、そのドキュメントを参照して下さい。

See Also
^^^^^^^^

* :command:`define_property`
* :command:`set_property`
