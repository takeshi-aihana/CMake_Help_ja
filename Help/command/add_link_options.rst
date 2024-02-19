add_link_options
----------------

.. versionadded:: 3.13

実行可能なファイル、共有ライブラリ、モジュール・ライブラリのリンク・オプションを追加する。

.. code-block:: cmake

  add_link_options(<option> ...)

このコマンドを使ってリンク時のオプションを追加できますが、同等の機能がライブラリを追加するコマンド（:command:`target_link_libraries` や :command:`link_libraries` コマンド）にもあります。
:prop_dir:`ディレクトリ <LINK_OPTIONS>` や  :prop_tgt:`ターゲット <LINK_OPTIONS>` 向けの ``LINK_OPTIONS`` というプロパティをそれぞれ参照して下さい。

.. note::

  このコマンドは静的ライブラリのオプションを追加することはできない（このターゲットはリンカを使用しないため）。
  静的ライブラリを作成するためのアーカイバや MSVC 系ライブラリのフラグを追加する方法は、:prop_tgt:`STATIC_LIBRARY_OPTIONS` というターゲット・プロパティを参照のこと。

.. |command_name| replace:: ``add_link_options``
.. include:: GENEX_NOTE.txt

.. include:: DEVICE_LINK_OPTIONS.txt

.. include:: OPTIONS_SHELL.txt

.. include:: LINK_OPTIONS_LINKER.txt

参考情報
^^^^^^^^

* :command:`link_libraries`
* :command:`target_link_libraries`
* :command:`target_link_options`

* CMake 変数の :variable:`CMAKE_<LANG>_FLAGS` と :variable:`CMAKE_<LANG>_FLAGS_<CONFIG>` はコンパイラの全ての呼び出しに渡されるプログラミング言語別のフラグを追加できる
  （これらのフラグには、コンパイラやリンカ向けのフラグも含まれる）。

