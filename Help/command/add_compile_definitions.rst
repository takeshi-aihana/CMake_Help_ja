add_compile_definitions
-----------------------

.. versionadded:: 3.12

プリプロセッサの定義をソース・ファイルのコンパイルに追加する。

.. code-block:: cmake

  add_compile_definitions(<definition> ...)

プリプロセッサの定義の ``<definition> ...`` をコンパイラのコマンドラインに追加します。

現在の ``CMakeLists.txt`` ファイルの :prop_dir:`COMPILE_DEFINITIONS` というディレクトリ・プロパティに ``<definition> ...`` を追加します。
また、現在の ``CMakeLists.txt`` ファイルにある全てのビルド・ターゲットの :prop_tgt:`COMPILE_DEFINITIONS` というターゲット・プロパティにも ``<definition> ...`` を追加します。

プリプロセッサの定義は、``VAR`` または ``VAR=value`` のような構文にして下さい。
関数スタイルの定義はサポートしていません。
CMake は、ネイティブのビルドシステムに対してプリプロセッサの定義を自動的にエスケープします（CMake 言語の構文は、一部のプリプロセッサ定義を扱う際にエスケープが必要になる場合があるので注意して下さい）。

.. versionadded:: 3.26
  プリプロセッサ定義の先頭にある ``-D`` は追加せずに削除するようになった。

.. |command_name| replace:: ``add_compile_definitions``
.. include:: GENEX_NOTE.txt

参考情報
^^^^^^^^

* :command:`target_compile_definitions` コマンドはターゲット（アーキテクチャ）専用の定義を追加する。
