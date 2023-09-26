.. cmake-manual-description: CMake コマンドライン・リファレンス

cmake(1)
********

概要
====

.. parsed-literal::

 `プロジェクトのビルドシステムを生成する`_
  cmake [<options>] -B <path-to-build> [-S <path-to-source>]
  cmake [<options>] <path-to-source | path-to-existing-build>

 `プロジェクトをビルドする`_
  cmake --build <dir> [<options>] [-- <build-tool-options>]

 `プロジェクトをインストールする`_
  cmake --install <dir> [<options>]

 `プロジェクトを開く`_
  cmake --open <dir>

 `CMake スクリプトを実行する`_
  cmake [-D <var>=<value>]... -P <cmake-script-file>

 `コマンドライン・ツールを実行する`_
  cmake -E <command> [<options>]

 `パッケージ検索ツールを実行する`_
  cmake --find-package [<options>]

 `ワークフローのプリセットを実行する`_
  cmake --workflow [<options>]

 `ヘルプを表示する`_
  cmake --help[-<topic>]

説明
====

:program:`cmake` は、「クロスプラットフォームに対応したビルドシステムのジェネレータ（生成器）」である CMake のコマンドライン・インタフェース（CLI）です。上の `概要`_ に一覧にした操作は、この下の各セクションで説明するように実行することが可能です。

CMake でプロジェクトをビルドする場合は, `プロジェクトのビルドシステムを生成する`_ を参照して下さい。追加で　:program:`cmake` を使って `プロジェクトをビルドする`_ とか、 `プロジェクトをインストールする`_ ことができる他、関連する他のビルドツール（``make`` など）を直接実行できます。また :program:`cmake` を使って `ヘルプを表示する`_  こともできます。

その他の操作として、CMake スクリプトを作成するソフトウェア開発者が使用することを前提として、彼らのビルドを CMake でサポートするために :manual:`CMake language <cmake-language(7)>` があります。

:program:`cmake` コマンドライン・インタフェースの代わりとして利用できるグラフィカル・ユーザ・インタフェース（GUI）については :manual:`ccmake <ccmake(1)>` と :manual:`cmake-gui <cmake-gui(1)>` を参照して下さい。CMake から実行できる単体テストやパッケージ作成機能に対するコマンドライン・インタフェースについては、それぞれ :manual:`ctest <ctest(1)>` と :manual:`cpack <cpack(1)>` を参照して下さい。

CMake 全体の詳細については、このマニュアルの最後にある  `関連項目`_ のリンクを参照して下さい。


CMake のビルドシステムについて
==============================

*ビルドシステム* とはプロジェクトの実行形式やライブラリをソースコードから生成する方法を *ビルド・ツール* を使って自動化を含めて具体化するものです。
それは、たとえば、あるビルドシステムはコマンドラインの ``make`` と ``Makefile`` であったり、あるいは統合開発環境（IDE）で使うプロジェクトファイルであったりします。
いろいろな種類のビルドシステムを何度も保守することにならないように、あるプロジェクトでは :manual:`CMake language <cmake-language(7)>` で説明しているファイルを使い、ビルドシステムを抽象的に指定している場合があります。
そのような場合、CMake はこれらのファイルから  *ジェネレータ* と呼ばれるバックエンドを介し、ユーザが好むビルドシステムを生成してくれます。

CMake でビルドシステムを生成する場合、以下を選択する必要があります：

ソースツリー（Source Tree）
  プロジェクトによって提供されたソースファイルなどが配置されているトップレベルのディレクトリには、マニュアルの :manual:`cmake-language(7)` に記載されている ``CMakeLists.txt`` というファイルを配置し、その中にビルドシステムを指定します。
  このファイルは複数のサブディレクトリに配置でき、その場合は :manual:`cmake-buildsystem(7)` で説明されているように、ビルド・ターゲットやそのれらの依存関係をそれぞれ指定します。

ビルドツリー（Build Tree）
  トップレベルのディレクトリには、生成されたビルドシステムのファイルと、それを使ってビルドした結果（たとえば、実行形式やライブラリ）が格納されます。
  CMake は、このディレクトリをビルドツリーとして認識してビルドシステムで利用する構成情報やビルドオプションを ``CMakeCache.txt`` というファイルに保存します。

  ソースツリーの初期状態を維持しておくため、それとは別のビルドツリーを使い、いわゆる「ソースの外（*out-of-source*）」でビルドを実施します。
  ビルドツリーがソースツリーの中にあるような「ソースの中（*in-source*）」でのビルドもサポートしていますが、これは推奨していません。

ジェネレータ（Generator）
  これは CMake で生成するビルドシステムの種類を指定します。
  サポートしている全てのジェネレータについては、マニュアルの :manual:`cmake-generators(7)` を参照して下さい。
  :option:`cmake --help` を実行した場合も、実際に利用可能なジェネレータの一覧が表示されます。
  オプションの :option:`-G <cmake -G>` を使ってジェネレータを指定する、あるいは現在のプラットフォーム向けのデフォルトのジェネレータを CMake に選択させる方法があります。

  :ref:`Command-Line Build Tool Generators` の中から選択すると、CMake はコンパイラなどツールチェインに必要な環境がすでに Shell の中で構築されているものとみなします。
  あるいは :ref:`IDE Build Tool Generators` の中から選択すると特定の環境は必要ありません。


.. _`プロジェクトのビルドシステムを生成する`:

プロジェクトのビルドシステムを生成する
======================================

次に示すコマンド・シグネチャのいずれかに、ソースツリーとビルドツリーを指定して CMake を実行すると、ビルドシステムが生成されます：

``cmake [<options>] -B <path-to-build> [-S <path-to-source>]``

  .. versionadded:: 3.13

  ``<path-to-build>`` にはビルドツリーのパス名、``<path-to-source>`` にソースツリーのパス名を指定します。
  ここで指定するパスは現在の作業ディレクトリ（cwd : Current Working directory）からの絶対パスまたは相対パスです。
  ソースツリーには ``CMakeLists.txt`` ファイルが配置されている必要があります。
  ビルドツリーは、実行時に存在していなければ自動的に生成されます。
  実行例は:

  .. code-block:: console

    $ cmake -S src -B build

``cmake [<options>] <path-to-source>``
  ビルドツリーは現在の作業ディレクトリ（cwd）とし、``<path-to-source>`` にソースツリーのパス名を指定します。
  ここで指定するパスは現在の作業ディレクトリ（cwd）からの絶対パスまたは相対パスです。
  ソースツリーには ``CMakeLists.txt`` ファイルが配置されている必要がありますが、 ``CMakeCache.txt`` ファイルは *配置しないで下さい* （このファイルでビルドツリーを識別するため）。
  実行例は:

  .. code-block:: console

    $ mkdir build ; cd build
    $ cmake ../src

``cmake [<options>] <path-to-existing-build>``
  ``<path-to-existing-build>`` にはビルドツリーのパス名を指定し、ソースツリーは、そのパスに配置されている（前の CMake 実行で生成した）既存の ``CMakeCache.txt`` ファイルから取得します。
  ここで指定するパスは現在の作業ディレクトリ（cwd）からの絶対パスまたは相対パスです。
  実行例は:

  .. code-block:: console

    $ cd build
    $ cmake .

全てのコマンド・シグネチャにおいて、``<options>`` には 0 個以上の `オプション`_ を指定します。

上で説明したとおり、ソースツリーとビルドツリーを一緒に指定しても問題ありません。
オプションの :option:`-S <cmake -S>` や :option:`-B <cmake -B>` で指定するパス名は常にソースツリーまたはビルドツリーとして扱われます。
単純に引数として指定したツリーのパス名は、その場所と前に指定したツリーのパスの種類（ソースツリーまたはビルドツリー）に応じて扱われます。
ソースツリーまたはビルドツリーのうちパス名を一つしか与えなかった場合、現在の作業ディレクトリ（cwd）がもう一方のツリーのパス名として使用されます。
整理すると:

============================== ============ ===========
 コマンドライン                ソースツリー ビルドツリー
============================== ============ ===========
 ``cmake -B build``             `cwd`        ``build``
 ``cmake -B build src``         ``src``      ``build``
 ``cmake -B build -S src``      ``src``      ``build``
 ``cmake src``                  ``src``      `cwd`
 ``cmake build`` (既存)         `loaded`     ``build``
 ``cmake -S src``               ``src``      `cwd`
 ``cmake -S src build``         ``src``      ``build``
 ``cmake -S src -B build``      ``src``      ``build``
============================== ============ ===========

.. versionchanged:: 3.23

  複数のソースツリーを指定した場合、CMake は警告を出します。
  このような指定はドキュメントで明記していませんし未サポートですが、古いバージョンの CMake は誤って警告を出さず、最後に指定したパス名をそのまま使うものがあります。
  いずれにせよ、複数のソースツリーを引数として渡さないようにして下さい。

ビルドシステムを生成したら、これに対応するネィティブなビルドツールを使ってプロジェクトをビルドします。
たとえば、:generator:`Unix Makefiles` のジェネレータを使った場合、ビルドシステムでそのまま ``make`` を実行できます：

  .. code-block:: console

    $ make
    $ make install

もしくは、ビルドシステムが :program:`cmake` を使用した場合は、自動的に `プロジェクトをビルドする`_ に従い、適切なビルドツールを呼び出してくれます。

.. _`CMake Options`:

オプション
----------

.. program:: cmake

.. include:: OPTIONS_BUILD.txt

.. option:: --fresh

 .. versionadded:: 3.24

 ビルドツリーで、新たな構成を作成する。
 これにより、既存の ``CMakeCache.txt`` ファイルと関連する ``CMakeFiles/`` ディレクトリが削除され、改めて実行した構成で再作成される。

.. option:: -L[A][H]


 `ADVANCED` ではないキャッシュ変数を一覧表示する。

 CMake の ``CACHE`` エントリ のうち ``INTERNAL`` にも :prop_cache:`ADVANCED` にも分類されていない全ての変数を一覧表示する。
 これは CMake で、オプション :option:`-D <cmake -D>` を使ってカスタマイズ可能な設定の現在の値を確認する際に便利なオプションである。
 任意の変数の変更すると、たくさんの変数が生成される場合がある。
 ``A`` を指定すると :prop_cache:`ADVANCED` な変数も表示する。
 ``H`` を指定すると各変数のヘルプも表示する

.. option:: -N

 CMake をビューモードで実行する。

 キャッシュを読み込むだけ。
 ビルドシステムの設定（*configure*）や生成（*generate*）のステップは実行しない。


.. option:: --graphviz=<file>

 ビルド時の依存関係を図化した graphviz のグラフを生成する（詳細は :module:`CMakeGraphVizOptions` を参照のこと）。

 生成した graphviz 向けの入力ファイルには、CMake プロジェクトの中にある全てのライブラリと実行形式の依存関係が含まれる。
 詳細は :module:`CMakeGraphVizOptions` のドキュメントを参照のこと。

.. option:: --system-information [file]

 お使いのシステムの情報をダンプする。

 現在のシステムについて幅広く情報を収集しダンプする。
 CMake プロジェクトのビルド・ツリーの最上位で実行すると、キャシュやログファイルなどの追加情報もダンプする。

.. option:: --log-level=<level>

 ログ・レベルを ``<level>`` にする。

 :command:`message` コマンドは、ここで指定したログ・レベル以上のメッセージだけ出力する。
 指定可能なログ・レベルは： ``ERROR``, ``WARNING``, ``NOTICE``, ``STATUS`` （これがデフォルト）, ``VERBOSE``, ``DEBUG``, ``TRACE``

 CMake を実行するたびに同じログ・レベルを使いたいのであれば、このオプションの代わりに変数 :variable:`CMAKE_MESSAGE_LOG_LEVEL` にセットしてキャッシュ変数にすること。
 もしオプションと環境変数の両方を指定した場合は、このオプションが優先される。

 後方互換性の理由から、``--loglevel`` オプションも同義として扱われる。

 .. versionadded:: 3.25
   :ref:`現在のログ・レベルを問い合わせる <query_message_log_level>` 方法については、:command:`cmake_language` コマンドの説明を参照のこと。

.. option:: --log-context

 :command:`message` コマンドで出力するメッセージにコンテキストを付与する。

 このオプションは、現在の ``CMakeLists.txt`` に対する CMake 実行時にのみコンテキストを表示するようにする。
 CMake を実行するたびにコンテキストを表示させたいのであれば、環境変数 :variable:`CMAKE_MESSAGE_CONTEXT_SHOW` にセットしてキャッシュ変数にすること。
 なの、このオプションを指定して CMake を実行すると、環境変数 :variable:`CMAKE_MESSAGE_CONTEXT_SHOW` の値は無視される。

.. option:: --debug-trycompile

 :command:`try_compile` / :command:`try_run` コマンドの呼び出しで生成したファイルやディレクトリを削除しない。
 これは CMake 実行時のエラーをデバッグする際に便利なオプションである。

 :command:`try_compile` コマンドは同じビルドツリーで実行しても特に警告しないので、もしプロジェクトが :command:`try_compile` コマンドを複数回呼び出すようになっていると、期待した結果にならない可能性があるので注意すること。
 このような場合、たとえば一つ前の実行で生成された成果物によりから別のテストの結果が変わってしまい、最終的な結果が変わってしまう可能性がある。
 このオプションはデバッグ時にのみ使用すべきである。

 （上のケースに関して、 :command:`try_run` コマンドは実際のところ :command:`try_compile` コマンドと等価である。
 これら二つのコマンドの組み合わせが、ここで説明した潜在的な問題の影響を受ける可能性があるので注意すること）

 .. versionadded:: 3.25

   このオプションを指定すると :command:`try_compile` コマンドを実行するたびに、 チェックしたディレクトリに関するログ・メッセージを出力する。

.. option:: --debug-output

 CMake をデバッグモードで実行する。

 CMake 実行中に、 :command:`message(SEND_ERROR)` コマンドを使ってスタックトレースなどの追加情報を出力する。

.. option:: --debug-find

 CMake の `find` コマンドをデバッグモードで実行する。

 CMake 実行中に `find` コマンドの追加情報を標準エラー出力に出力する。
 この出力は可読なフォーマットであり、出力結果の解析に向いた出力ではないので注意すること。
 プロジェクトでさらにローカルな部分をデバッグする際は、環境変数 :variable:`CMAKE_FIND_DEBUG_MODE` の説明も参照してください。

.. option:: --debug-find-pkg=<pkg>[,...]

 CMake の :command:`find_package(\<pkg\>) <find_package>` （ ``<pkg>`` はパッケージ名を表す（大小文字を区別する）文字列をカンマで区切って並べたもの）コマンドをデバッグモードで実行する。

 :option:`--debug-find <cmake --debug-find>` オプションと違う点は、指定したパッケージについてのみ検索すること。

.. option:: --debug-find-var=<var>[,...]


 変数 ``<var>`` （変数を表す文字列をカンマで区切って並べたもの）を検索する CMake の `find` コマンドをデバッグモードで実行する。

 :option:`--debug-find <cmake --debug-find>` オプションと違う点は、指定した変数についてのみ検索すること。

.. option:: --trace

 CMake をトレースモードで実行する。

 何のコマンドが何処で呼ばれたかが分かるトレース情報を出力する。

.. option:: --trace-expand

 CMake をトレースモードで実行する。

 :option:`--trace <cmake --trace>` オプションと違う点は、変数の内容が展開されて出力されること。

.. option:: --trace-format=<format>

 CMake を出力フォーマットを指定して、トレースモードで実行する。

 ``<format>`` には次のいずれかを指定すること：

   ``human``
     一行ごとに可読なフォーマットでトレース結果を出力する。
     これがデフォルトのフォーマット。

   ``json-v1``
     一行毎に JSON ドキュメントとしてトレース結果を出力する。
     一行毎に JSON ドキュメントは改行 ( ``\n`` ) で区切られる。
     JSON ドキュメントの内に改行文字が出力されないことが保証されている。

     .. code-block:: json
       :caption: JSON ドキュメント形式のトレース出力

       {
         "file": "/full/path/to/the/CMake/file.txt",
         "line": 0,
         "cmd": "add_executable",
         "args": ["foo", "bar"],
         "time": 1579512535.9687231,
         "frame": 2,
         "global_frame": 4
       }

     出力されるメンバのキー：

     ``file``
       呼び出された関数が定義された CMake ソースファイルへの絶対パス

     ``line``
       呼び出された関数定義の先頭行

     ``line_end``
       呼び出された関数定義の終了行（関数が一行だけの場合は空、このメンバは ``json-v1`` のバージョン 2 で追加された）

     ``defer``
       :command:`cmake_language(DEFER)` コマンドで関数呼び出しが延期された時に追加されるメンバで、延期された呼び出しを識別する文字列 ``<id>`` 

     ``cmd``
      呼び出された関数名

     ``args``
       関数の引数を表す文字列を要素とするリスト

     ``time``
       関数呼出しが発生した時のタイムスタンプ（エポックからの秒数）

     ``frame``
       処理中の ``CMakeLists.txt`` のコンテキスト中で、関数が呼び出されたスタックフレームの深さ

     ``global_frame``
       ``CMakeLists.txt`` ファイル全体で追跡される関数が呼び出されたスタックフレームの深さ（このメンバは ``json-v1`` のバージョン 2 で追加された）

     さらに、最初に出力される JSON ドキュメントには、次のフォーマットに従った ``version`` キーが含まれる：
     
     .. code-block:: json
       :caption: JSON のバージョン出力

       {
         "version": {
           "major": 1,
           "minor": 2
         }
       }

     出力されるメンバのキー：

     ``version``
       JSON フォーマットのバージョンを表す（このバージョンは major と minor のメンバを持つ）

.. option:: --trace-source=<file>

 CMake をトレースモードで実行するが、指定したファイルに対してだけ出力する。

 このオプションは複数指定できる（複数のファイルを指定できる）。

.. option:: --trace-redirect=<file>

 CMake をトレースモードで実行し、その出力を標準エラー出力ではなくファイルに書き出す。

.. option:: --warn-uninitialized

 初期化していない値を警告する。

 初期化していない変数を使用した時に警告を出力する。

.. option:: --warn-unused-vars

 何もしない。
 CMake バージョン 3.2 以下では、このオプションは未使用の変数について警告を出力する。
 CMake バージョン 3.3 から 3.18 では、このオプションは機能しない。
 CMake バージョン 3.19 以上では、このオプションは削除された。

.. option:: --no-warn-unused-cli

 コマンドライン・オプションについては警告しない。

 コマンドラインから変数を追加したが実際に使用していない場合、`find` コマンドの検索対象にしない。

.. option:: --check-system-vars

 ビルドシステムのファイルに、未使用または未初期化の変数がないかチェックする。

 デフォルトでは未使用と未初期化の変数は :variable:`CMAKE_SOURCE_DIR` と :variable:`CMAKE_BINARY_DIR` の中でしか検索しない。
 このオプションを指定すると、それ以外のファイルについてチェックし警告するようになる。

.. option:: --compile-no-warning-as-error

 ビルドターゲットのプロパティ :prop_tgt:`COMPILE_WARNING_AS_ERROR` と変数 :variable:`CMAKE_COMPILE_WARNING_AS_ERROR` を無視し、ターゲットのコンパイル時に警告がエラーとして扱われないようにする。

.. option:: --profiling-output=<path>

 オプション :option:`--profiling-format <cmake --profiling-format>` と組み合わせて使用する。指定したパスに結果を出力する。

.. option:: --profiling-format=<format>

 CMake スクリプトに対するプロファイルデータを、指定したフォーマットで出力する。

 このオプションは、CMake スクリプト実行時のパフォーマンス分析に役立つ。
 この出力を、サードパーティ製のアプリケーションを使って可読でわかりやすいフォーマットに変換する必要がある。

 現在サポートしているフォーマット：
 ``google-trace`` は Google トレース・フォーマットで出力する（Google Chrome ブラウザの about:tracing タブやトレースコンパスのようなツールのプラグインを使ってパースできる）

.. option:: --preset <preset>, --preset=<preset>

 ``<path-to-source>/CMakePresets.json`` と ``<path-to-source>/CMakeUserPresets.json`` から :manual:`preset <cmake-presets(7)>` を読み込む。
 この preset には、ジェネレータとビルドツリー、変数のリスト、そして CMake に渡すその他の引数を指定できる。
 preset ファイルは、現在の作業ディレクトリ（cwd）に格納しておくこと。
 :manual:`CMake GUI <cmake-gui(1)>` は ``CMakePresets.json`` と ``CMakeUserPresets.json`` のファイルも認識できる。
 これらのファイルについて詳細は :manual:`cmake-presets(7)` を参照のこと。

 preset は、他のコマンドライン・オプションよりも先に CMake が解釈する。
 preset が指定したオプション（変数、ジェネレータなど）は全て、コマンドラインから指定した値で上書きすることが可能。
 たとえば、preset が変数  ``MYVAR`` に ``1`` をセットしている場合に、コマンドラインから :option:`-D <cmake -D>` オプションで ``2`` にセットすると、この ``2`` が優先される。

.. option:: --list-presets[=<type>]

 ``<type>`` の preset で利用可能なものの一覧を出力する。
 ``<type>`` に指定できる値は ``configure`` 、``build`` 、 ``test`` 、 ``package`` 、 ``all`` のいずれか。
 ``<type>`` を省略すると ``configure`` が指定されたものとする。
 preset ファイルは、現在の作業ディレクトリ（cwd）に格納しておくこと。

.. option:: --debugger

  CMake 言語を使った対話型デバッガを有効にする。
  CMake は :option:`--debugger-pipe <cmake --debugger-pipe>` オプションで指定したパイプ上にデバッグ用インタフェースを公開する。
  このインタフェースは `Debug Adapter Protocol`_  に、以下に示す幾つか変更を加えた仕様に準拠する。

  ``initialize`` コマンドの応答にはデバッグ中の CMake のバージョンを表す ``cmakeVersion`` と云う追加フィールドが含まれている。

  .. code-block:: json
    :caption: デバッガの initialize コマンドの応答

    {
      "cmakeVersion": {
        "major": 3,
        "minor": 27,
        "patch": 0,
        "full": "3.27.0"
      }
    }

  このフィールドのメンバは：

  ``major``
    メジャーバージョンの番号（整数値）

  ``minor``
    マイナーバージョンの番号（整数値）

  ``patch``
    パッチバージョンの番号（整数値）

  ``full``
    バージョンの完全形（文字列）

.. _`Debug Adapter Protocol`: https://microsoft.github.io/debug-adapter-protocol/

.. option:: --debugger-pipe <pipe name>, --debugger-pipe=<pipe name>

  デバッガとの通信で使用するパイプの名前（Windows 系）またはドメインソケット（Unix 系）を指定する。

.. option:: --debugger-dap-log <log path>, --debugger-dap-log=<log path>

  デバッガとの通信ログをファイルに記録する。

.. _`Build Tool Mode`:

プロジェクトをビルドする
========================

.. program:: cmake

CMake はプロジェクトで生成済みのビルドシステム（ビルドツリー）をビルドするためのマンドライン（*CLI Signature*）を提供している：

.. code-block:: shell

  cmake --build <dir>             [<options>] [-- <build-tool-options>]
  cmake --build --preset <preset> [<options>] [-- <build-tool-options>]

これは、以下のオプションを使ってネイティブなビルドツールのコマンドライン・インタフェースを抽象化している：

.. option:: --build <dir>

  ビルドするプロジェクトのビルドツリー（バイナリツリー）。
  これはビルドに必須のディレクトリで、（preset が指定されていない限り）必ず最初に指定すること。

.. program:: cmake--build

.. option:: --preset <preset>, --preset=<preset>

  preset を使用して、いろいろなビルド・オプションを一度に指定する。
  プロジェクトのビルドツリー（ビルドディレクトリ）は ``configurePreset`` キーから推測する。
  preset ファイルは、現在の作業ディレクトリ（cwd）に格納しておくこと。
  詳細は :manual:`preset <cmake-presets(7)>` を参照のこと。

.. option:: --list-presets

  preset で利用可能なものの一覧を出力する。
  preset ファイルは、現在の作業ディレクトリ（cwd）に格納しておくこと。

.. option:: -j [<jobs>], --parallel [<jobs>]

  .. versionadded:: 3.12

  ビルド時で使用する並列実行が可能なジョブの最大数。
  ``<jobs>`` を省略すると、ネイティブなビルドツールのデフォルトの並列レベルを使用する。

  このオプションを指定せず、環境変数 :envvar:`CMAKE_BUILD_PARALLEL_LEVEL` がセットされている場合は、この環境変数の値がデフォルトの並列レベルとなる。

  一部のネイティブなビルドツールは常に並列ビルドを行う。
  ``<jobs>`` を ``1`` にすると、そのようなビルドツールでも単一のジョブに制限できる。

.. option:: -t <tgt>..., --target <tgt>...

  デフォルトのターゲットの代わりに ``<tgt>`` をビルドする。
  複数の ``<tgt>`` を指定する場合は空白で区切ること。

.. option:: --config <cfg>

  複数ある configuration ツールの中から ``<cfg>`` のツールを選択する。

.. option:: --clean-first

  まずターゲットを ``clean`` してからビルドする（ターゲットを ``clean`` したいだけなら、:option:`--target clean <cmake--build --target>` を使うこと）。

.. option:: --resolve-package-references=<value>

  .. versionadded:: 3.23

  ビルドを開始する前に、外部のパッケージ・マネージャ（例: Nuget）からリモートにあるパッケージ参照を解決する。
  ``<value>`` が ``on`` （デフォルト）の場合、パッケージはビルド開始する前にパッケージを復元する。
  ``<value>`` が ``only`` の場合、パッケージは復元されるが、ビルドは実行しない。
  ``<value>`` が ``off`` の場合、パッケージは復元されない。

  ターゲットがパッケージの参照を定義していない場合、このオプションは何もしない。

  この設定は preset （``resolvePackageReferences`` キー）で指定できる。
  このオプションを指定すると、preset の値は無視される。

  このオプションが指定されず、preset も提供されていない場合、ビルド環境固有のキャッシュ変数が評価され、パッケージを復元するかどうかを決定する。

  Visual Studio のジェネレータでは、パッケージ参照はプロパティ :prop_tgt:`VS_PACKAGE_REFERENCES` で定義される。
  その時のパッケージ参照は NuGet を使用して復元される。
  変数 ``CMAKE_VS_NUGET_PACKAGE_RESTORE`` を ``OFF`` にすると、この機能を無効にできる。

.. option:: --use-stderr

  このオプションは無視する。CMake のバージョンが 3.0 以上から、これ（標準エラー出力を使う）がデフォルトの挙動である。

.. option:: -v, --verbose

  ビルド冗長な出力を有効にする（もしネイティブのビルドツールがサポートしていれば、実行するビルドコマンドでも冗長な出力にする）

  環境変数 :envvar:`VERBOSE` またはキャシュ変数 :variable:`CMAKE_VERBOSE_MAKEFILE` がセットされている場合、このオプションを省略できる。

.. option:: --

  これより後ろにあるオプションをネイティブのビルドツールに渡す。

オプションを付けずに :option:`cmake --build` を実行すると表示される簡易ヘルプが。


プロジェクトをインストールする
==============================

.. program:: cmake

CMake provides a command-line signature to install an already-generated
project binary tree:

.. code-block:: shell

  cmake --install <dir> [<options>]

This may be used after building a project to run installation without
using the generated build system or the native build tool.
The options are:

.. option:: --install <dir>

  Project binary directory to install. This is required and must be first.

.. program:: cmake--install

.. option:: --config <cfg>

  For multi-configuration generators, choose configuration ``<cfg>``.

.. option:: --component <comp>

  Component-based install. Only install component ``<comp>``.

.. option:: --default-directory-permissions <permissions>

  Default directory install permissions. Permissions in format ``<u=rwx,g=rx,o=rx>``.

.. option:: --prefix <prefix>

  Override the installation prefix, :variable:`CMAKE_INSTALL_PREFIX`.

.. option:: --strip

  Strip before installing.

.. option:: -v, --verbose

  Enable verbose output.

  This option can be omitted if :envvar:`VERBOSE` environment variable is set.

Run :option:`cmake --install` with no options for quick help.

プロジェクトを開く
==================

.. program:: cmake

.. code-block:: shell

  cmake --open <dir>

Open the generated project in the associated application.  This is only
supported by some generators.


.. _`Script Processing Mode`:

CMake スクリプトを実行する
==========================

.. program:: cmake

.. code-block:: shell

  cmake [-D <var>=<value>]... -P <cmake-script-file> [-- <unparsed-options>...]

.. program:: cmake-P

.. option:: -D <var>=<value>

 Define a variable for script mode.

.. program:: cmake

.. option:: -P <cmake-script-file>

 Process the given cmake file as a script written in the CMake
 language.  No configure or generate step is performed and the cache
 is not modified.  If variables are defined using ``-D``, this must be
 done before the ``-P`` argument.

Any options after ``--`` are not parsed by CMake, but they are still included
in the set of :variable:`CMAKE_ARGV<n> <CMAKE_ARGV0>` variables passed to the
script (including the ``--`` itself).


.. _`Run a Command-Line Tool`:

コマンドライン・ツールを実行する
================================

.. program:: cmake

CMake provides builtin command-line tools through the signature

.. code-block:: shell

  cmake -E <command> [<options>]

.. option:: -E [help]

  Run ``cmake -E`` or ``cmake -E help`` for a summary of commands.

.. program:: cmake-E

Available commands are:

.. option:: capabilities

  .. versionadded:: 3.7

  Report cmake capabilities in JSON format. The output is a JSON object
  with the following keys:

  ``version``
    A JSON object with version information. Keys are:

    ``string``
      The full version string as displayed by cmake :option:`--version <cmake --version>`.
    ``major``
      The major version number in integer form.
    ``minor``
      The minor version number in integer form.
    ``patch``
      The patch level in integer form.
    ``suffix``
      The cmake version suffix string.
    ``isDirty``
      A bool that is set if the cmake build is from a dirty tree.

  ``generators``
    A list available generators. Each generator is a JSON object with the
    following keys:

    ``name``
      A string containing the name of the generator.
    ``toolsetSupport``
      ``true`` if the generator supports toolsets and ``false`` otherwise.
    ``platformSupport``
      ``true`` if the generator supports platforms and ``false`` otherwise.
    ``supportedPlatforms``
      .. versionadded:: 3.21

      Optional member that may be present when the generator supports
      platform specification via :variable:`CMAKE_GENERATOR_PLATFORM`
      (:option:`-A ... <cmake -A>`).  The value is a list of platforms known to
      be supported.
    ``extraGenerators``
      A list of strings with all the :ref:`Extra Generators` compatible with
      the generator.

  ``fileApi``
    Optional member that is present when the :manual:`cmake-file-api(7)`
    is available.  The value is a JSON object with one member:

    ``requests``
      A JSON array containing zero or more supported file-api requests.
      Each request is a JSON object with members:

      ``kind``
        Specifies one of the supported :ref:`file-api object kinds`.

      ``version``
        A JSON array whose elements are each a JSON object containing
        ``major`` and ``minor`` members specifying non-negative integer
        version components.

  ``serverMode``
    ``true`` if cmake supports server-mode and ``false`` otherwise.
    Always false since CMake 3.20.

  ``tls``
    .. versionadded:: 3.25

    ``true`` if TLS support is enabled and ``false`` otherwise.

  ``debugger``
    .. versionadded:: 3.27

    ``true`` if the :option:`--debugger <cmake --debugger>` mode
    is supported and ``false`` otherwise.

.. option:: cat [--] <files>...

  .. versionadded:: 3.18

  Concatenate files and print on the standard output.

  .. program:: cmake-E_cat

  .. option:: --

    .. versionadded:: 3.24

    Added support for the double dash argument ``--``. This basic implementation
    of ``cat`` does not support any options, so using a option starting with
    ``-`` will result in an error. Use ``--`` to indicate the end of options, in
    case a file starts with ``-``.

.. program:: cmake-E

.. option:: chdir <dir> <cmd> [<arg>...]

  Change the current working directory and run a command.

.. option:: compare_files [--ignore-eol] <file1> <file2>

  Check if ``<file1>`` is same as ``<file2>``. If files are the same,
  then returns ``0``, if not it returns ``1``.  In case of invalid
  arguments, it returns 2.

  .. program:: cmake-E_compare_files

  .. option:: --ignore-eol

    .. versionadded:: 3.14

    The option implies line-wise comparison and ignores LF/CRLF differences.

.. program:: cmake-E

.. option:: copy <file>... <destination>, copy -t <destination> <file>...

  Copy files to ``<destination>`` (either file or directory).
  If multiple files are specified, or if ``-t`` is specified, the
  ``<destination>`` must be directory and it must exist. If ``-t`` is not
  specified, the last argument is assumed to be the ``<destination>``.
  Wildcards are not supported. ``copy`` does follow symlinks. That means it
  does not copy symlinks, but the files or directories it point to.

  .. versionadded:: 3.5
    Support for multiple input files.

  .. versionadded:: 3.26
    Support for ``-t`` argument.

.. option:: copy_directory <dir>... <destination>

  Copy content of ``<dir>...`` directories to ``<destination>`` directory.
  If ``<destination>`` directory does not exist it will be created.
  ``copy_directory`` does follow symlinks.

  .. versionadded:: 3.5
    Support for multiple input directories.

  .. versionadded:: 3.15
    The command now fails when the source directory does not exist.
    Previously it succeeded by creating an empty destination directory.

.. option:: copy_directory_if_different <dir>... <destination>

  .. versionadded:: 3.26

  Copy changed content of ``<dir>...`` directories to ``<destination>`` directory.
  If ``<destination>`` directory does not exist it will be created.

  ``copy_directory_if_different`` does follow symlinks.
  The command fails when the source directory does not exist.

.. option:: copy_if_different <file>... <destination>

  Copy files to ``<destination>`` (either file or directory) if
  they have changed.
  If multiple files are specified, the ``<destination>`` must be
  directory and it must exist.
  ``copy_if_different`` does follow symlinks.

  .. versionadded:: 3.5
    Support for multiple input files.

.. option:: create_symlink <old> <new>

  Create a symbolic link ``<new>`` naming ``<old>``.

  .. versionadded:: 3.13
    Support for creating symlinks on Windows.

  .. note::
    Path to where ``<new>`` symbolic link will be created has to exist beforehand.

.. option:: create_hardlink <old> <new>

  .. versionadded:: 3.19

  Create a hard link ``<new>`` naming ``<old>``.

  .. note::
    Path to where ``<new>`` hard link will be created has to exist beforehand.
    ``<old>`` has to exist beforehand.

.. option:: echo [<string>...]

  Displays arguments as text.

.. option:: echo_append [<string>...]

  Displays arguments as text but no new line.

.. option:: env [<options>] [--] <command> [<arg>...]

  .. versionadded:: 3.1

  Run command in a modified environment. Options are:

  .. program:: cmake-E_env

  .. option:: NAME=VALUE

    Replaces the current value of ``NAME`` with ``VALUE``.

  .. option:: --unset=NAME

    Unsets the current value of ``NAME``.

  .. option:: --modify ENVIRONMENT_MODIFICATION

    .. versionadded:: 3.25

    Apply a single :prop_test:`ENVIRONMENT_MODIFICATION` operation to the
    modified environment.

    The ``NAME=VALUE`` and ``--unset=NAME`` options are equivalent to
    ``--modify NAME=set:VALUE`` and ``--modify NAME=unset:``, respectively.
    Note that ``--modify NAME=reset:`` resets ``NAME`` to the value it had
    when :program:`cmake` launched (or unsets it), not to the most recent
    ``NAME=VALUE`` option.

  .. option:: --

    .. versionadded:: 3.24

    Added support for the double dash argument ``--``. Use ``--`` to stop
    interpreting options/environment variables and treat the next argument as
    the command, even if it start with ``-`` or contains a ``=``.

.. program:: cmake-E

.. option:: environment

  Display the current environment variables.

.. option:: false

  .. versionadded:: 3.16

  Do nothing, with an exit code of 1.

.. option:: make_directory <dir>...

  Create ``<dir>`` directories.  If necessary, create parent
  directories too.  If a directory already exists it will be
  silently ignored.

  .. versionadded:: 3.5
    Support for multiple input directories.

.. option:: md5sum <file>...

  Create MD5 checksum of files in ``md5sum`` compatible format::

     351abe79cd3800b38cdfb25d45015a15  file1.txt
     052f86c15bbde68af55c7f7b340ab639  file2.txt

.. option:: sha1sum <file>...

  .. versionadded:: 3.10

  Create SHA1 checksum of files in ``sha1sum`` compatible format::

     4bb7932a29e6f73c97bb9272f2bdc393122f86e0  file1.txt
     1df4c8f318665f9a5f2ed38f55adadb7ef9f559c  file2.txt

.. option:: sha224sum <file>...

  .. versionadded:: 3.10

  Create SHA224 checksum of files in ``sha224sum`` compatible format::

     b9b9346bc8437bbda630b0b7ddfc5ea9ca157546dbbf4c613192f930  file1.txt
     6dfbe55f4d2edc5fe5c9197bca51ceaaf824e48eba0cc453088aee24  file2.txt

.. option:: sha256sum <file>...

  .. versionadded:: 3.10

  Create SHA256 checksum of files in ``sha256sum`` compatible format::

     76713b23615d31680afeb0e9efe94d47d3d4229191198bb46d7485f9cb191acc  file1.txt
     15b682ead6c12dedb1baf91231e1e89cfc7974b3787c1e2e01b986bffadae0ea  file2.txt

.. option:: sha384sum <file>...

  .. versionadded:: 3.10

  Create SHA384 checksum of files in ``sha384sum`` compatible format::

     acc049fedc091a22f5f2ce39a43b9057fd93c910e9afd76a6411a28a8f2b8a12c73d7129e292f94fc0329c309df49434  file1.txt
     668ddeb108710d271ee21c0f3acbd6a7517e2b78f9181c6a2ff3b8943af92b0195dcb7cce48aa3e17893173c0a39e23d  file2.txt

.. option:: sha512sum <file>...

  .. versionadded:: 3.10

  Create SHA512 checksum of files in ``sha512sum`` compatible format::

     2a78d7a6c5328cfb1467c63beac8ff21794213901eaadafd48e7800289afbc08e5fb3e86aa31116c945ee3d7bf2a6194489ec6101051083d1108defc8e1dba89  file1.txt
     7a0b54896fe5e70cca6dd643ad6f672614b189bf26f8153061c4d219474b05dad08c4e729af9f4b009f1a1a280cb625454bf587c690f4617c27e3aebdf3b7a2d  file2.txt

.. option:: remove [-f] <file>...

  .. deprecated:: 3.17

  Remove the file(s). The planned behavior was that if any of the
  listed files already do not exist, the command returns a non-zero exit code,
  but no message is logged. The ``-f`` option changes the behavior to return a
  zero exit code (i.e. success) in such situations instead.
  ``remove`` does not follow symlinks. That means it remove only symlinks
  and not files it point to.

  The implementation was buggy and always returned 0. It cannot be fixed without
  breaking backwards compatibility. Use ``rm`` instead.

.. option:: remove_directory <dir>...

  .. deprecated:: 3.17

  Remove ``<dir>`` directories and their contents. If a directory does
  not exist it will be silently ignored.
  Use ``rm`` instead.

  .. versionadded:: 3.15
    Support for multiple directories.

  .. versionadded:: 3.16
    If ``<dir>`` is a symlink to a directory, just the symlink will be removed.

.. option:: rename <oldname> <newname>

  Rename a file or directory (on one volume). If file with the ``<newname>`` name
  already exists, then it will be silently replaced.

.. option:: rm [-rRf] [--] <file|dir>...

  .. versionadded:: 3.17

  Remove the files ``<file>`` or directories ``<dir>``.
  Use ``-r`` or ``-R`` to remove directories and their contents recursively.
  If any of the listed files/directories do not exist, the command returns a
  non-zero exit code, but no message is logged. The ``-f`` option changes
  the behavior to return a zero exit code (i.e. success) in such
  situations instead. Use ``--`` to stop interpreting options and treat all
  remaining arguments as paths, even if they start with ``-``.

.. option:: sleep <number>

  .. versionadded:: 3.0

  Sleep for ``<number>`` seconds. ``<number>`` may be a floating point number.
  A practical minimum is about 0.1 seconds due to overhead in starting/stopping
  CMake executable. This can be useful in a CMake script to insert a delay:

  .. code-block:: cmake

    # Sleep for about 0.5 seconds
    execute_process(COMMAND ${CMAKE_COMMAND} -E sleep 0.5)

.. option:: tar [cxt][vf][zjJ] file.tar [<options>] [--] [<pathname>...]

  Create or extract a tar or zip archive.  Options are:

  .. program:: cmake-E_tar

  .. option:: c

    Create a new archive containing the specified files.
    If used, the ``<pathname>...`` argument is mandatory.

  .. option:: x

    Extract to disk from the archive.

    .. versionadded:: 3.15
      The ``<pathname>...`` argument could be used to extract only selected files
      or directories.
      When extracting selected files or directories, you must provide their exact
      names including the path, as printed by list (``-t``).

  .. option:: t

    List archive contents.

    .. versionadded:: 3.15
      The ``<pathname>...`` argument could be used to list only selected files
      or directories.

  .. option:: v

    Produce verbose output.

  .. option:: z

    Compress the resulting archive with gzip.

  .. option:: j

    Compress the resulting archive with bzip2.

  .. option:: J

    .. versionadded:: 3.1

    Compress the resulting archive with XZ.

  .. option:: --zstd

    .. versionadded:: 3.15

    Compress the resulting archive with Zstandard.

  .. option:: --files-from=<file>

    .. versionadded:: 3.1

    Read file names from the given file, one per line.
    Blank lines are ignored.  Lines may not start in ``-``
    except for ``--add-file=<name>`` to add files whose
    names start in ``-``.

  .. option:: --format=<format>

    .. versionadded:: 3.3

    Specify the format of the archive to be created.
    Supported formats are: ``7zip``, ``gnutar``, ``pax``,
    ``paxr`` (restricted pax, default), and ``zip``.

  .. option:: --mtime=<date>

    .. versionadded:: 3.1

    Specify modification time recorded in tarball entries.

  .. option:: --touch

    .. versionadded:: 3.24

    Use current local timestamp instead of extracting file timestamps
    from the archive.

  .. option:: --

    .. versionadded:: 3.1

    Stop interpreting options and treat all remaining arguments
    as file names, even if they start with ``-``.

  .. versionadded:: 3.1
    LZMA (7zip) support.

  .. versionadded:: 3.15
    The command now continues adding files to an archive even if some of the
    files are not readable.  This behavior is more consistent with the classic
    ``tar`` tool. The command now also parses all flags, and if an invalid flag
    was provided, a warning is issued.

.. program:: cmake-E

.. option:: time <command> [<args>...]

  Run ``<command>`` and display elapsed time (including overhead of CMake frontend).

  .. versionadded:: 3.5
    The command now properly passes arguments with spaces or special characters
    through to the child process. This may break scripts that worked around the
    bug with their own extra quoting or escaping.

.. option:: touch <file>...

  Creates ``<file>`` if file do not exist.
  If ``<file>`` exists, it is changing ``<file>`` access and modification times.

.. option:: touch_nocreate <file>...

  Touch a file if it exists but do not create it.  If a file does
  not exist it will be silently ignored.

.. option:: true

  .. versionadded:: 3.16

  Do nothing, with an exit code of 0.

Windows-specific Command-Line Tools
-----------------------------------

The following ``cmake -E`` commands are available only on Windows:

.. option:: delete_regv <key>

  Delete Windows registry value.

.. option:: env_vs8_wince <sdkname>

  .. versionadded:: 3.2

  Displays a batch file which sets the environment for the provided
  Windows CE SDK installed in VS2005.

.. option:: env_vs9_wince <sdkname>

  .. versionadded:: 3.2

  Displays a batch file which sets the environment for the provided
  Windows CE SDK installed in VS2008.

.. option:: write_regv <key> <value>

  Write Windows registry value.


パッケージ検索ツールを実行する
==============================

.. program:: cmake--find-package

CMake provides a pkg-config like helper for Makefile-based projects:

.. code-block:: shell

  cmake --find-package [<options>]

It searches a package using :command:`find_package()` and prints the
resulting flags to stdout.  This can be used instead of pkg-config
to find installed libraries in plain Makefile-based projects or in
autoconf-based projects (via ``share/aclocal/cmake.m4``).

.. note::
  This mode is not well-supported due to some technical limitations.
  It is kept for compatibility but should not be used in new projects.

.. _`Workflow Mode`:

ワークフローのプリセットを実行する
==================================

.. program:: cmake

:manual:`CMake Presets <cmake-presets(7)>` provides a way to execute multiple
build steps in order:

.. code-block:: shell

  cmake --workflow [<options>]

The options are:

.. option:: --workflow

  Select a :ref:`Workflow Preset` using one of the following options.

.. program:: cmake--workflow

.. option:: --preset <preset>, --preset=<preset>

  Use a workflow preset to specify a workflow. The project binary directory
  is inferred from the initial configure preset. The current working directory
  must contain CMake preset files.
  See :manual:`preset <cmake-presets(7)>` for more details.

.. option:: --list-presets

  Lists the available workflow presets. The current working directory must
  contain CMake preset files.

.. option:: --fresh

  Perform a fresh configuration of the build tree.
  This removes any existing ``CMakeCache.txt`` file and associated
  ``CMakeFiles/`` directory, and recreates them from scratch.

ヘルプを表示する
================

.. program:: cmake

To print selected pages from the CMake documentation, use

.. code-block:: shell

  cmake --help[-<topic>]

with one of the following options:

.. include:: OPTIONS_HELP.txt

To view the presets available for a project, use

.. code-block:: shell

  cmake <source-dir> --list-presets


.. _`CMake Exit Code`:

Return Value (Exit Code)
========================

Upon regular termination, the :program:`cmake` executable returns the exit code ``0``.

If termination is caused by the command :command:`message(FATAL_ERROR)`,
or another error condition, then a non-zero exit code is returned.


関連項目
========

.. include:: LINKS.txt
