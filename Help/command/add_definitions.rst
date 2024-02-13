add_definitions
---------------

コンパイル時に ``-D`` の定義フラグを追加する。

.. code-block:: cmake

  add_definitions(-DFOO -DBAR ...)

``CMakeLists.txt`` がある現在のディレクトリ内にあるターゲットと、後から追加したサブディレクトリ内のターゲットに対するコンパイラ・オプションに ``-DFOO -DBAR ...`` の定義フラグを追加します。
このコマンドは任意の定義フラグを追加する以外に、プリプロセッサ定義を追加する際にも利用できます。

.. note::

  このコマンドは、次のコマンドでそれぞれ置き換えられた：

  * プリプロセッサ定義を追加する場合は、:command:`add_compile_definitions` コマンドを使うこと。
  * インクルード・ディレクトリを追加する場合は :command:`include_directories` コマンドを使うこと。
  * その他のオプションを追加する場合は :command:`add_compile_options` コマンドを使うこと。

``-D`` や ``/D`` で始まるフラグは、``CMakeLists.txt`` がある現在のディレクトリの :prop_dir:`COMPILE_DEFINITIONS` プロパティに自動的に追加されます。
これにより（CMake の下位互換性の理由から）まれに重要な値を含んだ定義フラグが変換されずに残されてしまう場合があります。
特定のスコープやビルド構成で、プリプロセッサ定義を追加する方法について詳細については、:prop_dir:`ディレクトリ <COMPILE_DEFINITIONS>` や :prop_tgt:`ターゲット <COMPILE_DEFINITIONS>` や  :prop_sf:`ソース・ファイル <COMPILE_DEFINITIONS>` の ``COMPILE_DEFINITIONS`` プロパティのドキュメントを参照して下さい。

参考情報
^^^^^^^^

* ビルドシステムのプロパティ定義について詳細は :manual:`cmake-buildsystem(7)` を参照のこと。
