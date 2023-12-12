find_library
------------

.. |FIND_XXX| replace:: ``find_library``
.. |NAMES| replace:: NAMES name1 [name2 ...] [NAMES_PER_DIR]
.. |SEARCH_XXX| replace:: ライブラリ
.. |SEARCH_XXX_DESC| replace:: ライブラリの絶対パス
.. |prefix_XXX_SUBDIR| replace:: ``<prefix>/lib``
.. |entry_XXX_SUBDIR| replace:: ``<entry>/lib``

.. |FIND_XXX_REGISTRY_VIEW_DEFAULT| replace:: ``TARGET``

.. |FIND_PACKAGE_ROOT_PREFIX_PATH_XXX| replace::
   CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合は ``<prefix>/lib/<arch>`` と、|FIND_PACKAGE_ROOT_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_PREFIX_PATH_XXX| replace::
   CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合は ``<prefix>/lib/<arch>`` と、|CMAKE_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_XXX_PATH| replace:: :variable:`CMAKE_LIBRARY_PATH`
.. |CMAKE_XXX_MAC_PATH| replace:: :variable:`CMAKE_FRAMEWORK_PATH`

.. |ENV_CMAKE_PREFIX_PATH_XXX| replace::
   CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合は ``<prefix>/lib/<arch>`` と、|ENV_CMAKE_PREFIX_PATH_XXX_SUBDIR|
.. |ENV_CMAKE_XXX_PATH| replace:: :envvar:`CMAKE_LIBRARY_PATH`
.. |ENV_CMAKE_XXX_MAC_PATH| replace:: :envvar:`CMAKE_FRAMEWORK_PATH`

.. |SYSTEM_ENVIRONMENT_PATH_XXX| replace:: 環境変数の ``LIB`` と ``PATH`` にセットされているディレクトリ
.. |SYSTEM_ENVIRONMENT_PATH_WINDOWS_XXX| replace::
   Windows 系プラットフォームで、CMake のバージョン 3.3〜3.27 は CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合、追加で次のディレクトリを検索していた: ``<prefix>/lib/<arch>`` と |SYSTEM_ENVIRONMENT_PREFIX_PATH_XXX_SUBDIR|
   （これは CMake バージョン 3.28 から削除される）

.. |CMAKE_SYSTEM_PREFIX_PATH_XXX| replace::
   CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合は ``<prefix>/lib/<arch>`` と |CMAKE_SYSTEM_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_SYSTEM_XXX_PATH| replace::
   :variable:`CMAKE_SYSTEM_LIBRARY_PATH`
.. |CMAKE_SYSTEM_XXX_MAC_PATH| replace::
   :variable:`CMAKE_SYSTEM_FRAMEWORK_PATH`

.. |CMAKE_FIND_ROOT_PATH_MODE_XXX| replace::
   :variable:`CMAKE_FIND_ROOT_PATH_MODE_LIBRARY`

.. include:: FIND_XXX.txt

このコマンドは ``NAMES`` オプションに複数の「|SEARCH_XXX|」を指定しても、デフォルトでは全てのディレクトリに対して一度に1つの |SEARCH_XXX| を探します。
一方、``NAMES_PER_DIR`` オプションは、一度に1つのディレクトリに対して全ての「|SEARCH_XXX|」をまとめて探すようコマンドに指示します。

``NAMES`` オプションに指定した |SEARCH_XXX| はそれぞれファイル名として、ホスト・プラットフォームに固有の接頭子（たとえば ``lib``）と接尾子（たとえば ``.so``）が付けられます。
したがって、たとえば ``libfoo.a`` のようなふぁいるめいを直接指定できます。
これにより UNIX 系プラットフォームで静的ライブラリを見つけることができます。

見つかったライブラリが「フレームワーク」の場合、``<VAR>`` にはフレームワークへの絶対パス ``<fullPath>/A.framework`` が格納されます。
フレームワークへの絶対パスが1個のライブラリを指す場合、CMake は ``-framework A`` と ``-F<fullPath>`` を使ってそのフレームワークをターゲットにリンクします。

.. versionadded:: 3.28

  見つかったライブラリを ``.xcframework`` フォルダにすることができるようになった。

CMake 変数の :variable:`CMAKE_FIND_LIBRARY_CUSTOM_LIB_SUFFIX` に任意の文字列をセットすると、検索対象のすべてのディレクトリの ``lib/`` を ``lib${CMAKE_FIND_LIBRARY_CUSTOM_LIB_SUFFIX}/`` で置き換えたディレクトリでライブラリを検索します。
この CMake 変数は、:prop_gbl:`FIND_LIBRARY_USE_LIB32_PATHS`、:prop_gbl:`FIND_LIBRARY_USE_LIBX32_PATHS`、そして :prop_gbl:`FIND_LIBRARY_USE_LIB64_PATHS` といったグローバルなプロパティを上書きします。

:prop_gbl:`FIND_LIBRARY_USE_LIB32_PATHS` というグローバルなプロパティに任意の文字列をセットすると、検索対象のすべてのディレクトリに ``32/`` を追加し、``lib/`` を ``lib32/`` で置き換えたディレクトリでライブラリを検索します。
このプロパティは、:command:`project` コマンドでサポートしているプログラミング言語の少なくとも1つが利用できる場合は、そのプラットフォームに対して自動的にセットされます（FIXME: 意味不明）。

:prop_gbl:`FIND_LIBRARY_USE_LIBX32_PATHS` というグローバルなプロパティに任意の文字列をセットすると、検索対象のすべてのディレクトリ ``x32/`` を追加し、 ``lib/`` を ``libx32/`` で置き換えたディレクトリでライブラリを検索します。
このプロパティは、:command:`project` コマンドでサポートしているプログラミング言語の少なくとも1つが利用できる場合は、そのプラットフォームに対して自動的にセットされます（FIXME: 意味不明）。

:prop_gbl:`FIND_LIBRARY_USE_LIB64_PATHS` というグローバルなプロパティに任意の文字列をセットすると、検索対象のすべてのディレクトリ ``64/`` を追加し、 ``lib/`` を ``lib64/`` で置き換えたディレクトリでライブラリを検索します。
このプロパティは、:command:`project` コマンドでサポートしているプログラミング言語の少なくとも1つが利用できる場合は、そのプラットフォームに対して自動的にセットされます（FIXME: 意味不明）。
