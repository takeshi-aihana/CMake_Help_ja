add_subdirectory
----------------

プロジェクトにサブディレクトリを追加する。

.. code-block:: cmake

  add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL] [SYSTEM])

プロジェクトのビルド・ツリーにサブディレクトリを追加します。
追加する ``source_dir`` には ``CMakeLists.txt`` とソース・ファイルがあるディレクトリを指定します。
``source_dir`` が相対パスの場合は :variable:`CMAKE_CURRENT_SOURCE_DIR` をベースディレクトリにして評価します。
``binary_dir`` にはファイルを出力するディレクトリを指定します。
``binary_dir`` が相対パスの場合は :variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリにして評価します。
``binary_dir`` を指定しない場合は、（相対パスを評価する前の） ``source_dir`` を使用します。
``source_dir`` にある ``CMakeLists.txt`` は、後続のコマンドで入力ファイルを処理する前に、CMake によって直ちに実行されます。

``EXCLUDE_FROM_ALL`` オプションを指定すると、追加したサブディレクトリのビルド・ターゲットは親ディレクトリの ``ALL`` ターゲットには含まれません（IDE プロジェクトのファイルから除外されます）。
したがって追加したサブディレクトリにあるターゲットは、ユーザが明示的にビルドする必要があります。
このオプションの目的は、例えばサンプル・プログラムのように有用ではあるが必須ではないものが、プロジェクトの別のディレクトリに格納されている場合に使用することです。
通常、追加したサブディレクトリでは、独自に :command:`project` コマンドを呼び出して完全なビルドシステム（Visual Studio IDE のソリューション・ファイル）が生成されるようにする必要があります。
Note that inter-target dependencies supersede this exclusion（FIXME: 意味不明）。
親ディレクトリでビルドしたターゲットが、ここで追加したサブディレクトリのビルド・ターゲットに依存するよう場合、その依存関係を解決するために、依存対象のビルド・ターゲットが親ディレクトリのプロジェクトのビルドシステムに自動的に追加されます。

.. versionadded:: 3.25
  ``SYSTEM`` オプションで、追加したサブディレクトリのディレクトリ・プロパティ :prop_dir:`SYSTEM` が ``TRUE`` にセットされるようになった。
  このプロパティは、そのサブディレクトリにあるビルド・ターゲットのターゲット・プロパティ :prop_tgt:`SYSTEM` を初期化するために使用する（まだ初期化されていないものだけ）。
