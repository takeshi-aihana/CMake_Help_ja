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

 `コマンド・ツールを実行する`_
  cmake -E <command> [<options>]

 `パッケージ検索ツールを実行する`_
  cmake --find-package [<options>]

 `ワークフローのプリセットを実行する`_
  cmake --workflow [<options>]

 `ヘルプを表示する`_
  cmake --help[-<topic>]

説明
====

:program:`cmake` は、「クロスプラットフォームに対応したビルドシステムを生成する」CMake のコマンドライン・インタフェース（CLI）です。上の `概要`_ に一覧にした操作は、この下の各セクションで説明するようにして実行できます。

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

  「:ref:`Command-Line Build Tool Generators`」の中から選択すると、CMake はコンパイラなどツールチェインに必要な環境がすでに Shell の中で構築されているものとみなします。
  あるいは「:ref:`IDE Build Tool Generators`」の中から選択すると特定の環境は必要ありません。


.. _`プロジェクトのビルドシステムを生成する`:

プロジェクトのビルドシステムを生成する
======================================

次に示すコマンドライン（*CLI Signature*）のいずれかに、ソースツリーとビルドツリーを指定して CMake を実行すると、ビルドシステムが生成されます：

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

全てのコマンドラインにおいて、``<options>`` には 0 個以上の `オプション`_ を指定します。

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
 ``H`` を指定すると各変数のヘルプも表示する。

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
 CMake プロジェクトのビルドツリーの最上位で実行すると、キャシュやログファイルなどの追加情報もダンプする。

.. option:: --log-level=<level>

 ログ・レベルを ``<level>`` にする。

 :command:`message` コマンドは、ここで指定したログ・レベル以上のメッセージだけ出力する。
 指定可能なログ・レベルは： ``ERROR``, ``WARNING``, ``NOTICE``, ``STATUS`` （これがデフォルト）, ``VERBOSE``, ``DEBUG``, ``TRACE``

 CMake を実行するたびに同じログ・レベルを使いたいのであれば、このオプションの代わりに変数 :variable:`CMAKE_MESSAGE_LOG_LEVEL` にセットしてキャッシュ変数にすること。
 もしオプションと環境変数の両方を指定した場合は、このオプションが優先される。

 後方互換性の理由から、``--loglevel`` オプションも同義として扱われる。

 .. versionadded:: 3.25
   「:ref:`現在のログ・レベルを問い合わせる <query_message_log_level>`」方法については、:command:`cmake_language` コマンドの説明を参照のこと。

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

 CMake をデバッグ・モードで実行する。

 CMake 実行中に、 :command:`message(SEND_ERROR)` コマンドを使ってスタックトレースなどの追加情報を出力する。

.. option:: --debug-find

 CMake の `find` コマンドをデバッグ・モードで実行する。

 CMake 実行中に `find` コマンドの追加情報を標準エラー出力に出力する。
 この出力は可読なフォーマットであり、出力結果の解析に向いた出力ではないので注意すること。
 プロジェクトでさらにローカルな部分をデバッグする際は、環境変数 :variable:`CMAKE_FIND_DEBUG_MODE` の説明も参照してください。

.. option:: --debug-find-pkg=<pkg>[,...]

 CMake の :command:`find_package(\<pkg\>) <find_package>` （ ``<pkg>`` はパッケージ名を表す（大小文字を区別する）文字列をカンマで区切って並べたもの）コマンドをデバッグ・モードで実行する。

 :option:`--debug-find <cmake --debug-find>` オプションと違う点は、指定したパッケージについてのみ検索すること。

.. option:: --debug-find-var=<var>[,...]


 変数 ``<var>`` （変数を表す文字列をカンマで区切って並べたもの）を検索する CMake の `find` コマンドをデバッグ・モードで実行する。

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
    パッチレベルの番号（整数値）

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

CMake は、プロジェクトで生成したビルドシステム（ビルドツリー）をビルドするためのコマンドライン（*CLI Signature*）を提供している：

.. code-block:: shell

  cmake --build <dir>             [<options>] [-- <build-tool-options>]
  cmake --build --preset <preset> [<options>] [-- <build-tool-options>]

このコマンドラインは、以下のオプションを使ってネイティブなビルドツールのコマンドライン・インタフェースを抽象化している：

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

  複数ある configuration ツールの中から ``<cfg>`` を選択する。

.. option:: --clean-first

  まずターゲットを ``clean`` してからビルドする（ターゲットを ``clean`` したいだけなら、:option:`--target clean <cmake--build --target>` を使うこと）。

.. option:: --resolve-package-references=<value>

  .. versionadded:: 3.23

  ビルドを開始する前に、外部のパッケージ・マネージャ（Nuget など）を使ってリモート・パッケージへの参照を解決する。
  ``<value>`` が ``on`` （デフォルト）の場合、ビルドを開始する前にパッケージを復元する。
  ``<value>`` が ``only`` の場合、パッケージは復元するがビルドは実行しない。
  ``<value>`` が ``off`` の場合、パッケージは復元しない。

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

  冗長な出力を有効にする（もしネイティブのビルドツールがサポートしていれば、実行するビルドコマンドでも冗長な出力にする）

  環境変数 :envvar:`VERBOSE` またはキャシュ変数 :variable:`CMAKE_VERBOSE_MAKEFILE` がセットされている場合、このオプションを省略できる。

.. option:: --

  これより後ろにあるオプションの並びをネイティブのビルドツールに渡す。

オプションを付けずに :option:`cmake --build` を実行すると簡易ヘルプを表示する。


プロジェクトをインストールする
==============================

.. program:: cmake

CMake は、プロジェクトで生成したビルドシステム（ビルドツリー）をインストールするためのコマンドライン（*CLI Signature*）を提供している：

.. code-block:: shell

  cmake --install <dir> [<options>]

このコマンドラインは、プロジェクトをビルドしたあと、生成したビルドシステムやネイティブなビルドツールを使わずにインストールを実行する際に使用する。
利用可能なオプションは：

.. option:: --install <dir>

  インストールするプロジェクトのビルドツリー（バイナリツリー）。
  これは必須のオプションで、一番最初に指定すること。

.. program:: cmake--install

.. option:: --config <cfg>

  複数ある configuration ツールの中から ``<cfg>`` を選択する。

.. option:: --component <comp>

  コンポーネント単位でインストールする。
  ``<comp>`` が指すコンポーネントだけインストールする。

.. option:: --default-directory-permissions <permissions>

  ディレクトリをインストールしたあとのパーミッション（これがデフォルトになる）。
  ``<u=rwx,g=rx,o=rx>`` のフォーマットで指定すること。

.. option:: --prefix <prefix>

  インストール先の Prefix を表す変数 :variable:`CMAKE_INSTALL_PREFIX` を上書きする。

.. option:: --strip

  インストールする前にバイナリをストリップする（バイナリに含まれるデバッグ情報などを削除する）。

.. option:: -v, --verbose

  冗長な出力を有効にする。

  環境変数 :envvar:`VERBOSE` がセットされている場合、このオプションを省略できる。

オプションを付けずに :option:`cmake --install` を実行すると簡易ヘルプを表示する。

プロジェクトを開く
==================

.. program:: cmake

.. code-block:: shell

  cmake --open <dir>

生成したプロジェクトのビルドシステムを、関連付けられたアプリケーションで開く。
この機能は一部のジェネレータでのみサポートしている。


.. _`Script Processing Mode`:

CMake スクリプトを実行する
==========================

.. program:: cmake

.. code-block:: shell

  cmake [-D <var>=<value>]... -P <cmake-script-file> [-- <unparsed-options>...]

.. program:: cmake-P

.. option:: -D <var>=<value>

 スクリプトに渡す変数を定義する。

.. program:: cmake

.. option:: -P <cmake-script-file>

 指定したファイルを CMake 言語で書かれたスクリプトとして処理する。
 何かを設定したり生成することはなく、キャッシュは変更されない。
 ``-D`` オプションで変数を定義する際は、この ``-P`` オプションを使った引数よりも前に置くこと。

CMake は ``--`` の後ろにあるオプションの並びを解析しないが、それらはスクリプトに渡される変数 :variable:`CMAKE_ARGV<n> <CMAKE_ARGV0>` の集合に ``--`` と一緒に含まれる。

.. _`Run a Command-Line Tool`:

コマンド・ツールを実行する
==========================

.. program:: cmake

CMake は、組み込みのコマンド・ツールを利用するためのコマンドライン（*CLI Signature*）を提供している：

.. code-block:: shell

  cmake -E <command> [<options>]

.. option:: -E [help]

  ``cmake -E`` または ``cmake -E help`` を実行すると利用可能なコマンド・ツールのサマリを表示する

.. program:: cmake-E

利用可能なコマンド・ツールは:

.. option:: capabilities

  .. versionadded:: 3.7

  CMake が処理できる機能と諸元を JSON 形式で報告する。
  出力される情報は以下のキーを持つ JSON オブジェクトである。

  ``version``
    バージョン情報を持つ JSON オブジェクトで、保持するキーは：

    ``string``
      CMake のオプション :option:`--version <cmake --version>` で表示されるバージョンの完全形（文字列）
    ``major``
      メジャーバージョンの番号（整数値）
    ``minor``
      マイナーバージョンの番号（整数値）
    ``patch``
      パッチレベルの番号（整数値）
    ``suffix``
      CMake のバージョン末尾の文字列
    ``isDirty``
      CMake ビルドが Dirty ツリーから行われるかどうかを示す論理値

  ``generators``
    利用可能なジェネレータを要素とするリスト。
    ジェネレータはそれぞれ以下のキーを持つ JSON オブジェクトである：

    ``name``
      ジェネレータの名前（文字列）
    ``toolsetSupport``
      このジェネレータがツールセットをサポートしている場合は ``true``、それ以外は ``false``
    ``platformSupport``
      このジェネレータが複数のプラットフォームをサポートしている場合は ``true``、それ以外は ``false``
    ``supportedPlatforms``
      .. versionadded:: 3.21

      このキーはオプションで、このジェネレータが変数 :variable:`CMAKE_GENERATOR_PLATFORM` （オプション :option:`-A ... <cmake -A>` ）経由で指定されたプラットフォーム仕様（ *Platform Specification* ）をサポートしている時に出力される場合がある。
      その値は、サポートしているプラットフォームを要素とするリスト。
    ``extraGenerators``
      このジェネレータと互換性がある全ての「:ref:`Extra Generators`」を表す文字列。

  ``fileApi``
    :manual:`cmake-file-api(7)` が利用可能な場合に追加される。
    その値は一個のメンバを持つ JSON オブジェクト：

    ``requests``
      サポートしている file-api の 0 個以上のリクエストを要素とする JSON 配列。
      各リクエストは次のメンバを持つ JSON オブジェクトである：

      ``kind``
        サポートしている「:ref:`file-api object kinds`」の一つを指定する。

      ``version``
        コンポーネントのバージョン（負ではない整数値）を表す ``major`` と ``minor`` を格納した JSON オブジェクトを要素とする JSON 配列。

  ``serverMode``
    CMake がサーバ・モードをサポートしている場合は ``true``、それ以外は ``false``。
    CMake バージョン 3.20 以降は常に ``false``。

  ``tls``
    .. versionadded:: 3.25

    TLS をサポートしている場合は ``true`` 、それ以外は ``false``。

  ``debugger``
    .. versionadded:: 3.27

    オプション :option:`--debugger <cmake --debugger>` をサポートしている場合は ``true``、それ以外は ``false``。

.. option:: cat [--] <files>...

  .. versionadded:: 3.18

  いくつかのファイルを連結して、その中身を標準出力に出力する。

  .. program:: cmake-E_cat

  .. option:: --

    .. versionadded:: 3.24

    二重ダッシュの引数 ``--`` をサポートした。
    この ``cat`` コマンドは基本的にオプションを受け取らない実装なので、 ``-`` で始まるオプションを渡すとエラーになる。
    ``-`` で始まるファイルを引数として渡す場合は、オプションではないことを CMake に指示するために ``--`` と併用すること。

.. program:: cmake-E

.. option:: chdir <dir> <cmd> [<arg>...]

  現在の作業ディレクトリ（cwd）を変更してからコマンドを実行する。

.. option:: compare_files [--ignore-eol] <file1> <file2>

  ``<file1>`` と ``<file2>`` が同じものかどうかをチェックする。
  両方が同じファイルならば ``0`` を返し、同じでなければ ``1`` を返す。
  それ以外（おかしな引数を渡した場合）は ``2`` を返す。

  .. program:: cmake-E_compare_files

  .. option:: --ignore-eol

    .. versionadded:: 3.14

    このオプションは行単位でファイルを比較する（その際は行末の LF/CRLF 文字は無視する）。

.. program:: cmake-E

.. option:: copy <file>... <destination>, copy -t <destination> <file>...

  指定したファイルを ``<destination>`` （ファイルまたはディレクトリのいずれか）にコピーする。
  複数のファイルを指定する、または ``-t`` を付ける場合、``<destination>`` にはディレクトリを指定すること（必ず存在しているディレクトリを指定すること）。
  ``-t`` を付けない場合、最後の引数が ``<destination>`` とみなす。
  ワイルドカードはサポートしていない。
  ``copy`` コマンドはシンボリックリンクをたどる。
  つまり、シンボリックリンクはコピーされず、シンボリックリンクが指すファイルまたはディレクトリをコピーする。

  .. versionadded:: 3.5
    複数のファイルを引数に指定できるようになった。

  .. versionadded:: 3.26
    ``-t`` オプションをサポートした。

.. option:: copy_directory <dir>... <destination>

  ディレクトリ ``<dir>...`` 配下をディレクトリ ``<destination>`` へコピーする。
  ``<destination>`` ディレクトリが存在していない場合は作成する。
  ``copy_directory`` コマンドはシンボリックリンクをたどる。

  .. versionadded:: 3.5
    複数のディレクトリを引数に指定できるようになった。

  .. versionadded:: 3.15
    指定したソース・ディレクトリ ``<dir>`` が存在しない場合は、コマンド実行を失敗するようになった。
    以前は、空の ``<destination>`` を作成して、コマンド実行を成功としていた。

.. option:: copy_directory_if_different <dir>... <destination>

  .. versionadded:: 3.26

  ディレクトリ ``<dir>...`` 配下で変更されたものをディレクトリ ``<destination>`` へコピーする。
  ``<destination>`` ディレクトリが存在していない場合は作成する。

  ``copy_directory_if_different`` コマンドはシンボリックリンクをたどる。
  指定したソース・ディレクトリ ``<dir>`` が存在しない場合は、コマンド実行は失敗する。

.. option:: copy_if_different <file>... <destination>

  指定したファイルのうち変更されているものを ``<destination>`` （ファイルまたはディレクトリのいずれか）にコピーする。
  複数のファイルを指定する場合、``<destination>`` にはディレクトリを指定すること（必ず存在しているディレクトリを指定すること）。
  ``copy_if_different`` コマンドはシンボリックリンクをたどる。

  .. versionadded:: 3.5
    複数のファイルを引数に指定できるようになった。

.. option:: create_symlink <old> <new>

  ``<old>`` を指すシンボリックリンク ``<new>`` を作成する。

  .. versionadded:: 3.13
    Windows でシンボリックリンクの作成をサポートした。

  .. note::
    作成するシンボリックリンク ``<new>`` までパスが事前に存在しているものとする。

.. option:: create_hardlink <old> <new>

  .. versionadded:: 3.19

  ``<old>`` を指すハードリンク ``<new>`` を作成する。

  .. note::
    作成するハードリンク ``<new>`` までのパスが事前に存在しているものとする。
    ``<old>`` には存在しているファイルを指定すること。

.. option:: echo [<string>...]

  引数を文字列として出力する。

.. option:: echo_append [<string>...]

  引数を文字列として出力するが、改行はしない。

.. option:: env [<options>] [--] <command> [<arg>...]

  .. versionadded:: 3.1

  指定したコマンドを、オプションで変更した環境で実行する。
  指定できるオプションは：

  .. program:: cmake-E_env

  .. option:: NAME=VALUE

    変数 ``NAME`` の値を ``VALUE`` にする。

  .. option:: --unset=NAME

    変数 ``NAME`` の値を解除して空にする。

  .. option:: --modify ENVIRONMENT_MODIFICATION

    .. versionadded:: 3.25

    変更した環境に :prop_test:`ENVIRONMENT_MODIFICATION` の操作を一度だけ適用する。

    オプションの ``NAME=VALUE`` と ``--unset=NAME`` は、それぞれ ``--modify NAME=set:VALUE`` と ``--modify NAME=unset:`` に等価である。
    ``--modify NAME=reset:`` は、``NAME=VALUE`` スタイルには従わず、変数 ``NAME`` を :program:`cmake` を実行したときの値（または何も値が設定されていない状態）にリセットするオプションであることに注意すること。

  .. option:: --

    .. versionadded:: 3.24

    二重ダッシュの引数 ``--`` をサポートした。
    ``--`` を指定すると、それ以降の引数をオプションや環境変数として扱うのを止め、``-`` で始まっていたり ``=`` を含んでいる場合でもコマンド ``<command>`` の引数 ``<arg>`` として扱う。

.. program:: cmake-E

.. option:: environment

  現在の環境変数を表示する。

.. option:: false

  .. versionadded:: 3.16

  終了コード 1 を返すだけで何もしない。

.. option:: make_directory <dir>...

  ディレクトリ ``<dir>`` を作成する。
  必要に応じて、親ディレクトリも作成する。
  既にディレクトリが存在している場合は何も出力せずに終了する。

  .. versionadded:: 3.5
    複数のディレクトリを引数に指定できるようになった。

.. option:: md5sum <file>...

  ファイルの MD5 チェックサムを ``md5sum`` と互換性のあるフォーマットで生成する::

     351abe79cd3800b38cdfb25d45015a15  file1.txt
     052f86c15bbde68af55c7f7b340ab639  file2.txt

.. option:: sha1sum <file>...

  .. versionadded:: 3.10

  ファイルの SHA1 チェックサムを ``sha1sum`` と互換性のあるフォーマットで生成する::

     4bb7932a29e6f73c97bb9272f2bdc393122f86e0  file1.txt
     1df4c8f318665f9a5f2ed38f55adadb7ef9f559c  file2.txt

.. option:: sha224sum <file>...

  .. versionadded:: 3.10

  ファイルの SHA224 チェックサムを ``sha224sum`` と互換性のあるフォーマットで生成する::

     b9b9346bc8437bbda630b0b7ddfc5ea9ca157546dbbf4c613192f930  file1.txt
     6dfbe55f4d2edc5fe5c9197bca51ceaaf824e48eba0cc453088aee24  file2.txt

.. option:: sha256sum <file>...

  .. versionadded:: 3.10

  ファイルの SHA256 チェックサムを ``sha256sum`` と互換性のあるフォーマットで生成する::

     76713b23615d31680afeb0e9efe94d47d3d4229191198bb46d7485f9cb191acc  file1.txt
     15b682ead6c12dedb1baf91231e1e89cfc7974b3787c1e2e01b986bffadae0ea  file2.txt

.. option:: sha384sum <file>...

  .. versionadded:: 3.10

  ファイルの SHA384 チェックサムを ``sha384sum`` と互換性のあるフォーマットで生成する::

     acc049fedc091a22f5f2ce39a43b9057fd93c910e9afd76a6411a28a8f2b8a12c73d7129e292f94fc0329c309df49434  file1.txt
     668ddeb108710d271ee21c0f3acbd6a7517e2b78f9181c6a2ff3b8943af92b0195dcb7cce48aa3e17893173c0a39e23d  file2.txt

.. option:: sha512sum <file>...

  .. versionadded:: 3.10

  ファイルの SHA512 チェックサムを ``sha512sum`` と互換性のあるフォーマットで生成する::

     2a78d7a6c5328cfb1467c63beac8ff21794213901eaadafd48e7800289afbc08e5fb3e86aa31116c945ee3d7bf2a6194489ec6101051083d1108defc8e1dba89  file1.txt
     7a0b54896fe5e70cca6dd643ad6f672614b189bf26f8153061c4d219474b05dad08c4e729af9f4b009f1a1a280cb625454bf587c690f4617c27e3aebdf3b7a2d  file2.txt

.. option:: remove [-f] <file>...

  .. deprecated:: 3.17

  指定したファイルを削除する。
  このコマンドの設計時の挙動は、指定したファイルが存在していなかったら 0 以外の値を終了コードとして返すが、ログは出力しないというものだった。
  そのような場合、``-f`` オプションは終了コードとして 0 を返すように挙動を変更する（すなわち成功する）。
  ``remove`` コマンドはシンボリックリンクをたどらない。
  つまり、シンボリックリンクだけ削除して、それが指しているファイルは削除しない。

  このコマンドの実装にはバグがあり、常に終了コードとして 0 を返していた。
  下位バージョンとの互換性を失わずに、このバグを修正することはできない懸念が残っていた。
  代わりに ``rm`` コマンドを使用すること。

.. option:: remove_directory <dir>...

  .. deprecated:: 3.17

  指定したディレクトリ ``<dir>`` 配下を削除する。
  指定したディレクトリが存在していなかったら何も出力せずに終了する。
  代わりに ``rm`` コマンドを使用すること。

  .. versionadded:: 3.15
    複数のディレクトリをサポートした。

  .. versionadded:: 3.16
    ``<dir>`` がディレクトリを指すシンボリックリンクの場合、シンボリックリンクだけを削除する。

.. option:: rename <oldname> <newname>

  一つのボリューム上にあるファイルやディレクトリの名前を変更する。
  既に ``<newname>`` の名前を持つファイルが存在している場合は何も出力せずに終了する。

.. option:: rm [-rRf] [--] <file|dir>...

  .. versionadded:: 3.17

  指定したファイル ``<file>`` やディレクトリ ``<dir>`` を削除する。
  オプションの ``-r`` や ``-R`` を使うと、ディレクトリの他にそのディレクトリ配下も再帰的に削除する。
  指定したファイルやディレクトリが存在していなかったら、 このコマンドは 0 以外の値を終了コードとして返すが、ログは出力しない。
  そのような場合、``-f`` オプションは終了コードとして 0 を返すように挙動を変更する（すなわち成功する）。
  二重ダッシュ ``--`` 以降はオプションとして扱うのを止めて、残りの引数は ``-`` で始まる場合でもすべてパス名として扱う。

.. option:: sleep <number>

  .. versionadded:: 3.0

  ``<number>`` 秒間スリープする。
  ``<number>`` は浮動小数点数で指定できる。
  実際のところ CMake の起動と停止でオーバーヘッドが発生するので、指定できる値は約 0.1 秒からである。
  これは、次に示すような CMake スクリプトの中で遅延を生成したい時に便利である：

  .. code-block:: cmake

    # 0.5秒間スリープする
    execute_process(COMMAND ${CMAKE_COMMAND} -E sleep 0.5)

.. option:: tar [cxt][vf][zjJ] file.tar [<options>] [--] [<pathname>...]

  tar や zip アーカイブを作成したり展開する。
  利用できるオプションは：

  .. program:: cmake-E_tar

  .. option:: c

    指定したファイルを格納した新しいアーカイブを作成する。
    ``<pathname>...`` は必須の引数である。

  .. option:: x

    アーカイブの中身を展開する。

    .. versionadded:: 3.15
      引数の ``<pathname>...`` を使用して、選択したファイルやディレクトリだけを展開することができる。
      選択したファイルやディレクトリを展開する際は、オプション ``-t`` で出力されるものと同じパス名を含む正確なファイル名を指定すること。

  .. option:: t

    アーカイブの中身（ファイルやディレクトリ）を一覧表示する。

    .. versionadded:: 3.15
      引数の ``<pathname>...`` を使用して、選択したファイルやディレクトリだけを一覧表示できるようになった。

  .. option:: v

    詳細な出力を生成する。

  .. option:: z

    作成したアーカイブを gzip で圧縮する。

  .. option:: j

    作成したアーカイブを bzip2 で圧縮する。

  .. option:: J

    .. versionadded:: 3.1

    作成したアーカイブを XZ で圧縮する。

  .. option:: --zstd

    .. versionadded:: 3.15

    作成したアーカイブを Zstandard で圧縮する。

  .. option:: --files-from=<file>

    .. versionadded:: 3.1

    指定した ``<file>`` からアーカイブに含めるファイル名を取得する（一行につき1ファイル）。
    空行は無視する。
    ``-`` で始まるファイルを追加するオプション ``--add-file=<name>`` を除いて、``<file>`` の各行は ``-`` で始めないこと。

  .. option:: --format=<format>

    .. versionadded:: 3.3

    作成するアーカイブのフォーマットを指定する。
    サポートしているフォーマット： ``7zip``, ``gnutar``, ``pax``, ``paxr`` （デフォルトは制限付き pax）, ``zip``

  .. option:: --mtime=<date>

    .. versionadded:: 3.1

    アーカイブの中の各ファイルで記録される変更日時（*modification time*）を指定する。

  .. option:: --touch

    .. versionadded:: 3.24

    アーカイブの中の各ファイルに記録されているタイムスタンプで展開する代わりに、現在のローカルの日時をタイムスタンプにする。

  .. option:: --

    .. versionadded:: 3.1

    これ以降の引数をオプションとして扱うのを止め、``-`` で始まる場合でも全てファイル名として扱う。

  .. versionadded:: 3.1
    LZMA (7zip) をサポートするようになった。

  .. versionadded:: 3.15
    一部のファイルを読み取れなかった場合でも停止せずに、アーカイブにファイルの追加を継続するようになった。
    この修正により、従来の ``tar`` ツールと同じ挙動をさらに保証できるようになった。
    さらに全てのフラグを解析するようになり、無効なフラグが指定されたら警告するようになった。

.. program:: cmake-E

.. option:: time <command> [<args>...]

  ``<command>`` を実行したあとに経過時間（CMake の起動と停止の時間も含む）を表示する

  .. versionadded:: 3.5
    空白や特殊文字を含む引数を子プロセスに正しく渡せるようになった。
    これにより、従来とおりに独自に追加した引用符やエスケープ文字でバグが発生しなくなったりスクリプトが機能しなくなる場合がある。

.. option:: touch <file>...

  ファイル ``<file>`` が存在していなければ、その ``<file>`` を作成する。
  ``<file>`` が存在している場合は、そのファイルのアクセス日時（*access time*）と変更日時（*modification time*）を変更する。

.. option:: touch_nocreate <file>...

  ``<file>`` が存在している場合はファイルを ``touch`` するが、``<file>`` は作成しない。
  ``<file>`` が存在していない場合は無視する。

.. option:: true

  .. versionadded:: 3.16

  終了コード 0 を返すだけで何もしない。

Windows 専用のコマンド・ツール
------------------------------

次の ``cmake -E`` コマンド・ツールは Windows でのみ利用できます：

.. option:: delete_regv <key>

  Windows レジストリで ``<key>`` の値を削除する。

.. option:: env_vs8_wince <sdkname>

  .. versionadded:: 3.2

  VS2005 にインストールされている Windows CE SDK 向けの環境を設定するバッチファイルを表示する。

.. option:: env_vs9_wince <sdkname>

  .. versionadded:: 3.2

  VS2008 にインストールされている Windows CE SDK 向けの環境を設定するバッチファイルを表示する。

.. option:: write_regv <key> <value>

  Windows レジストリで ``<key>`` の値に ``<value>`` を書き込む。


パッケージ検索ツールを実行する
==============================

.. program:: cmake--find-package

CMake は Makefile を使うプロジェクト向けに pkg-config に似たヘルパーを提供しています：

.. code-block:: shell

  cmake --find-package [<options>]

:command:`find_package()` コマンドを使ってライブラリを探し、見つかったフラグを標準出力に出力します。
これを pkg-config の代わりに使用すると、Makefile を使うプロジェクトや autoconf を使ったプロジェクトで（``share/aclocal/cmake.m4`` を介して）インストールされているライブラリを探すことができます。

.. note::
  このモードは、いくつかの技術的な制限のため十分にサポートされていません。
  これは CMake の互換性を維持するために残されている機能ですが、新しいプロジェクトでは使用しないで下さい。

.. _`Workflow Mode`:

ワークフローのプリセットを実行する
==================================

.. program:: cmake

:manual:`CMake Presets <cmake-presets(7)>` は複数のビルドステップを順番に実行していく手段を提供します。

.. code-block:: shell

  cmake --workflow [<options>]

利用できるオプションは：

.. option:: --workflow

  このあとのオプションのいずれかを使って「:ref:`Workflow Preset`」を選択する。

.. program:: cmake--workflow

.. option:: --preset <preset>, --preset=<preset>

  「:ref:`Workflow Preset`」を使ってワークフローを指定する。
  プロジェクトのビルドツリー（バイナリツリー）は、初期の設定プリセットから推測する。
  必ず、現在の作業ディレクトリ（cwd）に CMake のプリセットファイルを格納しておくこと。
  詳細は :manual:`preset <cmake-presets(7)>` を参照のこと。

.. option:: --list-presets

  利用可能なワークフローのプリセットを一覧表示する。
  必ず、現在の作業ディレクトリ（cwd）に CMake のプリセットファイルを格納しておくこと。

.. option:: --fresh

  ビルドツリーで、新たな構成を作成する。
  これにより、既存の ``CMakeCache.txt`` ファイルと関連する ``CMakeFiles/`` ディレクトリが削除され、改めて実行した構成で再作成される。

ヘルプを表示する
================

.. program:: cmake

CMake のドキュメントから特定のページを出力する場合は：

.. code-block:: shell

  cmake --help[-<topic>]

に、以下のオプションのいずれかを指定する：

.. include:: OPTIONS_HELP.txt

プロジェクトで利用可能なプリセットを表示する場合は：

.. code-block:: shell

  cmake <source-dir> --list-presets


.. _`CMake Exit Code`:

返り値（終了コード）
====================

正常で終了すると、:program:`cmake` は終了コードとして ``0`` を返します。

:command:`message(FATAL_ERROR)` を出力してで終了した、または別のエラーによって終了した場合は、終了コードとしてゼロ以外の数値を返します。


関連項目
========

.. include:: LINKS.txt
