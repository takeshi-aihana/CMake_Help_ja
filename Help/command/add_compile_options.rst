add_compile_options
-------------------

いろいろなオプションをソース・ファイルのコンパイルに追加する。

.. code-block:: cmake

  add_compile_options(<option> ...)

``<option> ...`` を :prop_dir:`COMPILE_OPTIONS` というディレクトリ・プロパティに追加します。
これにより、オプションは現在のディレクトリとその配下でターゲットをコンパイルする際に使用されます。

.. note::

  これらのオプションはターゲットのリンク時には使われない。
  これについては :command:`add_link_options` コマンドを参照のこと。

引数
^^^^

.. |command_name| replace:: ``add_compile_options``
.. include:: GENEX_NOTE.txt

.. include:: OPTIONS_SHELL.txt

サンプル
^^^^^^^^

コンパイラごとにサポートするオプションが異なるため、次のように、このコマンドはコンパイラ固有の条件ブロックで使用するのが一般的です：

.. code-block:: cmake

  if (MSVC)
      # warning level 4
      add_compile_options(/W4)
  else()
      # additional warnings
      add_compile_options(-Wall -Wextra -Wpedantic)
  endif()

プログラミング言語ごとにコンパイラのオプションを指定する場合は、:genex:`$<COMPILE_LANGUAGE>` や :genex:`$<COMPILE_LANGUAGE:languages>` などの :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を使って下さい。

参考情報
^^^^^^^^

* このコマンドを使って任意のコンパイラ・オプションを追加できる。
  ただし、プリプロセッサの定義を追加したディレクトリをインクルードする場合は、さらに特化したコマンドである :command:`add_compile_definitions` と :command:`include_directories` を使うことを推奨している。

* :command:`target_compile_options` コマンドはターゲット（アーキテクチャ）専用のオプションを追加する。

* このコマンドは全てのプログラミング言語に共通なコンパイラ・オプションを追加する。
  プログラミング言語ごとのコンパイラ・オプションを指定する場合は :genex:`COMPILE_LANGUAGE` というジェネレータ式を使うこと。

* ソース・ファイルのプロパティ :prop_sf:`COMPILE_OPTIONS` は一個のソース・ファイルにオプションを追加する。

* :command:`add_link_options` コマンドはリンクのオプションを追加する。

* CMake 変数の :variable:`CMAKE_<LANG>_FLAGS` と :variable:`CMAKE_<LANG>_FLAGS_<CONFIG>` は、コンパイラのすべての呼び出しに渡すプログラミング言語ごとのフラグを追加する。
  これにはコンパイルとリンクの両方のステージが含まれる。
