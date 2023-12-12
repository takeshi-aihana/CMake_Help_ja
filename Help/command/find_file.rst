find_file
---------

.. |FIND_XXX| replace:: ``find_file``
.. |NAMES| replace:: NAMES name1 [name2 ...]
.. |SEARCH_XXX| replace:: ファイル
.. |SEARCH_XXX_DESC| replace:: ファイルの絶対パス
.. |prefix_XXX_SUBDIR| replace:: ``<prefix>/include``
.. |entry_XXX_SUBDIR| replace:: ``<entry>/include``

.. |FIND_XXX_REGISTRY_VIEW_DEFAULT| replace:: ``TARGET``

.. |FIND_PACKAGE_ROOT_PREFIX_PATH_XXX| replace::
   CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合は ``<prefix>/include/<arch>`` と、|FIND_PACKAGE_ROOT_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_PREFIX_PATH_XXX| replace::
   CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合は ``<prefix>/include/<arch>`` と、|CMAKE_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_XXX_PATH| replace:: :variable:`CMAKE_INCLUDE_PATH`
.. |CMAKE_XXX_MAC_PATH| replace:: :variable:`CMAKE_FRAMEWORK_PATH`

.. |ENV_CMAKE_PREFIX_PATH_XXX| replace::
   CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合は ``<prefix>/include/<arch>`` と |ENV_CMAKE_PREFIX_PATH_XXX_SUBDIR|
.. |ENV_CMAKE_XXX_PATH| replace:: :envvar:`CMAKE_INCLUDE_PATH`
.. |ENV_CMAKE_XXX_MAC_PATH| replace:: :envvar:`CMAKE_FRAMEWORK_PATH`


.. |SYSTEM_ENVIRONMENT_PATH_XXX| replace:: 環境変数の ``INCLUDE`` と ``PATH`` にセットされているディレクトリ

.. |SYSTEM_ENVIRONMENT_PATH_WINDOWS_XXX| replace::
   Windows 系プラットフォームで、CMake のバージョン 3.3〜3.27 は CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合、追加で次のディレクトリを検索していた: ``<prefix>/include/<arch>`` と |SYSTEM_ENVIRONMENT_PREFIX_PATH_XXX_SUBDIR|
   （これは CMake バージョン 3.28 から削除される）

.. |CMAKE_SYSTEM_PREFIX_PATH_XXX| replace::
   CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE` がセットされている場合は ``<prefix>/include/<arch>`` と |CMAKE_SYSTEM_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_SYSTEM_XXX_PATH| replace::
   :variable:`CMAKE_SYSTEM_INCLUDE_PATH`
.. |CMAKE_SYSTEM_XXX_MAC_PATH| replace::
   :variable:`CMAKE_SYSTEM_FRAMEWORK_PATH`

.. |CMAKE_FIND_ROOT_PATH_MODE_XXX| replace::
   :variable:`CMAKE_FIND_ROOT_PATH_MODE_INCLUDE`

.. include:: FIND_XXX.txt
