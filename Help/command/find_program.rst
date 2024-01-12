find_program
------------

.. |FIND_XXX| replace:: ``find_program``
.. |NAMES| replace:: NAMES name1 [name2 ...] [NAMES_PER_DIR]
.. |SEARCH_XXX| replace:: プログラム
.. |SEARCH_XXX_DESC| replace:: プログラム
.. |prefix_XXX_SUBDIR| replace:: ``<prefix>/[s]bin``
.. |entry_XXX_SUBDIR| replace:: ``<entry>/[s]bin``

.. |FIND_XXX_REGISTRY_VIEW_DEFAULT| replace:: ``BOTH``

.. |FIND_PACKAGE_ROOT_PREFIX_PATH_XXX| replace::
   |FIND_PACKAGE_ROOT_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_PREFIX_PATH_XXX| replace::
   |CMAKE_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_XXX_PATH| replace:: :variable:`CMAKE_PROGRAM_PATH`
.. |CMAKE_XXX_MAC_PATH| replace:: :variable:`CMAKE_APPBUNDLE_PATH`

.. |ENV_CMAKE_PREFIX_PATH_XXX| replace::
   |ENV_CMAKE_PREFIX_PATH_XXX_SUBDIR|
.. |ENV_CMAKE_XXX_PATH| replace:: :envvar:`CMAKE_PROGRAM_PATH`
.. |ENV_CMAKE_XXX_MAC_PATH| replace:: :envvar:`CMAKE_APPBUNDLE_PATH`

.. |SYSTEM_ENVIRONMENT_PATH_XXX| replace:: ``PATH`` 自身のディレクトリ。
.. |SYSTEM_ENVIRONMENT_PATH_WINDOWS_XXX| replace:: \

.. |CMAKE_SYSTEM_PREFIX_PATH_XXX| replace::
   |CMAKE_SYSTEM_PREFIX_PATH_XXX_SUBDIR|
.. |CMAKE_SYSTEM_XXX_PATH| replace::
   :variable:`CMAKE_SYSTEM_PROGRAM_PATH`
.. |CMAKE_SYSTEM_XXX_MAC_PATH| replace::
   :variable:`CMAKE_SYSTEM_APPBUNDLE_PATH`

.. |CMAKE_FIND_ROOT_PATH_MODE_XXX| replace::
   :variable:`CMAKE_FIND_ROOT_PATH_MODE_PROGRAM`

.. include:: FIND_XXX.txt

``NAMES`` オプションに一つ以上の「名前」を指定した場合、このコマンドはデフォルトで全てのディレクトリに対して一個づつ名前を探していきます。
逆に ``NAMES_PER_DIR`` オプションは、一つのディレクトリに対して全ての名前を順番に探すよう指示します。
