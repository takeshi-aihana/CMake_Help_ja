ユーザ操作ガイド
****************

.. only:: html

   .. contents::

はじめに
========

あるソフトウェア・パッケージが、そのソースファイルと共に CMake ベースのビルドシステムを提供している場合、そのソフトウェアを利用する開発者はユーザ向けに用意された CMake ツールを実行してビルドする必要があります。

「行儀の良い」CMake ベースのビルドシステムは、ビルドした結果をソースディレクトリに出力するようなことはしません。そのため通常、ユーザはソースディレクトリの外でビルドを実行することになります。
まず CMake に適切なビルドシステムを生成するよう指示し、ビルド・ツールを呼び出してそのビルドシステムを処理します。
このビルドシステムはそれを生成したマシンに固有のものであり、再配布できるものではありません。
ソフトウェア・パッケージを利用する開発者は、それぞれ自分たちのシステムに固有のビルドシステムを CMake を使って生成する必要があります。

ここで生成したビルドシステムは、原則的に読み取り専用として扱って下さい。
「一時生成物（Primary Artifacts）」として CMake が生成したファイルはビルドシステムを定義するプロパティであり、ビルドシステムを生成した後に、たとえば IDE でそのようなプロパティを手動で設定する必要はありません。
CMake はビルドシステムを定期的に書き換えるので、ユーザがプロパティを変更しても上書きされてしまいます。

このマニュアルで説明している機能とユーザ・インタフェースは、すべての CMake ベースのビルドシステムで利用可能です。

CMake のツールで CMake が生成したファイルを処理している時、例えばコンパイラがサポートされていないとか、コンパイラが必要なコンパイル・オプションをサポートしていないとか、あるいは依存関係が見つからなかったなどのエラーをユーザに報告する場合があります。
これらのエラーは、別のコンパイラを選択するとか、:guide:`installing dependencies <Using Dependencies Guide>` とか、あるいは依存関係を見つける場所を CMake に指示するなどして、ユーザ自身で解決する必要があります。

コマンドライン cmake ツール
----------------------------

単純ですが、ソフトウェアの新しいコピーに対する :manual:`cmake(1)` の典型的な使い方は、まずビルド・ディレクトリを作成し、そのデレクトリで ``cmake`` を呼び出します：

.. code-block:: console

  $ cd some_software-1.4.2
  $ mkdir build
  $ cd build
  $ cmake .. -DCMAKE_INSTALL_PREFIX=/opt/the/prefix
  $ cmake --build .
  $ cmake --build . --target install

ソース・ディレクトリとは別のディレクトリでビルドすることが推奨されます。これによりソース・ディレクトリは初期の状態に保たれ、複数のツールチェインで同一のソースをビルドすることができ、ビルド・ディレクトリを削除するだけでビルド時の生成物を簡単にクリーンできます。

CMake のツールは、ソフトウェアの利用者ではなく、ソフトウェアを提供する開発者向けの警告をいろいろ報告する場合があります。
このような警告の多くは "This warning is for project developers" という注意書きが付いています。
このような警告は :option:`-Wno-dev <cmake -Wno-dev>` というフラグを :manual:`cmake(1)` に渡すことで無効にできます。

cmake-gui ツール
----------------

GUI インタフェースに慣れているユーザであれば :manual:`cmake-gui(1)` ツールを使い、CMake を呼び出してビルドシステムを生成することができます。

最初にソース・ディレクトリとビルド・ディレクトリを設定して下さい。
ソースとビルドには異なるディレクトリを指定することが推奨されます。

.. image:: GUI-Source-Binary.png
   :alt: Choosing source and binary directories

ビルドシステムの生成
====================

CMake のファイルから任意のビルドシステムを生成する時に、利用できるユーザ向けのツールがいくつかあります。
:manual:`ccmake(1)` と :manual:`cmake-gui(1)` といったツールは、いろいろ必要なオプション設定をユーザに提示してくれます。
:manual:`cmake(1)` は、コマンドラインからオプションを指定して呼び出せます。
このマニュアルではどのツールからでも設定できる共通のオプションについて説明していますが、オプションを設定するモードはツールごとに異なるので注意して下さい。

コマンドライン環境
------------------

``Makefiles`` または ``Ninja`` といったコマンドライン型のビルドシステムと一緒に :manual:`cmake(1)` を呼び出すときは、正しいビルド環境を指定して、正しいビルド・ツールが利用できることを保証する必要があります。
CMake は、必要に応じて適切な :variable:`build tool <CMAKE_MAKE_PROGRAM>` やコンパイラやリンカ、その他のツールを見つけられなければなりません。

Linux システムでは、大抵の場合、適切なツールが既にシステムにインストールされており、パッケージ・マネージャを使って簡単に追加することができるようになっています。
ユーザが独自にインストールしたり、デフォルトの場所以外にインストールしたその他のツールチェインも利用できます。

クロス・コンパイルする際、一部のプラットフォームでは環境変数を設定したり、環境を設定するためのスクリプトを用意する必要があるかもしれません。

Visual Studio には、コマンドライン型のビルドシステム向けに正しい環境を設定する複数のコマンドライン・プロンプトと ``vcvarsall.bat`` というスクリプトが同梱されています。 
Visual Studio のジェネレータを使う時に、必ず対応するコマンドライン環境を使用しなければならないわけではありませんが、そうすることにデメリットはありません。

Xcode を使う際、複数のバージョンの Xcode がインストールされているかもしれません。
どのバージョンを使うかはいろいろな方法で選択できますが、最も一般的な方法は次のとおりです：

* Xcode の IDE にある設定画面でデフォルトのバージョンを指定する
* コマンドライン・ツールである ``xcode-select`` を使ってデフォルトのバージョンを指定する
* CMake とビルド・ツールを実行する際に、環境変数の ``DEVELOPER_DIR`` にセットした値でデフォルトのバージョンを上書きする

なお :manual:`cmake-gui(1)` には環境変数の値を編集する機能があります。

コマンドラインの ``-G`` オプション
----------------------------------

CMake は、デフォルトでプラットフォームに基づいたジェネレータを選択します。
通常ユーザがソフトウェアをビルドする場合、デフォルトで選択されたジェネレータで十分です。

ユーザは :option:`-G <cmake -G>` というオプションでデフォルトのジェネレータを上書き指定できます：

.. code-block:: console

  $ cmake .. -G Ninja

:option:`cmake --help` を実行すると、ユーザが選択できる :manual:`generators <cmake-generators(7)>` の一覧が出力されます。
これらのジェネレータの名前は大小文字を区別するので注意して下さい。

Unix 系のシステム（含む Mac OS X）では :generator:`Unix Makefiles` がデフォルトのジェネレータです。
:generator:`NMake Makefiles` や :generator:`MinGW Makefiles` など、いろいろな派生型のジェネレータを Windows のさまざまな環境で利用することもできます。
これらのジェネレータは、``make`` や ``gmake`` や ``nmake``、あるいは同様のツールで解釈が可能な ``Makefile`` を生成します。
ジェネレータが対象としている環境やツールについて詳細は、それぞれのドキュメントを参照して下さい。

:generator:`Ninja` は主要なプラットフォームで利用可能なジェネレータの一つです。
``ninja`` は ``make`` と似たユース・ケースを持つビルド・ツールですが、パフォーマンスと処理効率を重視したものになっています。

Windows の場合、:manual:`cmake(1)` で Visual Studio IDE 向けのソリューションを生成できます。
Visual Studio のバージョンは、4桁の年を含む IDE の製品名で指定します。
エイリアスは、Visual C++ コンパイラの製品バージョンを表す2桁を指定する、あるいはそれらを組み合わせるなどして、Visual Studio のバージョンを参照する別の指定方として利用できます：

.. code-block:: console

  $ cmake .. -G "Visual Studio 2019"
  $ cmake .. -G "Visual Studio 16"
  $ cmake .. -G "Visual Studio 16 2019"

Visual Studio のジェネレータは、いろいろなアーキテクチャのターゲットをサポートしています。
:option:`-A <cmake -A>` オプションでターゲットのアーキテクチャを指定できます：

.. code-block:: console

  cmake .. -G "Visual Studio 2019" -A x64
  cmake .. -G "Visual Studio 16" -A ARM
  cmake .. -G "Visual Studio 16 2019" -A ARM64

Mac OS X の場合、:generator:`Xcode` というジェネレータを使って Xcode IDE 向けのプロジェクト・ファイルを生成します。

KDevelop4 や QtCreator や CLion のような一部の IDE は CMake ベースのビルドシステムをネイティブでサポートしています。

これらの IDE はジェネレータを選択するためのユーザ・インタフェースを提供しており、通常は ``Makefile`` から ``Ninja`` 系のジェネレータを選択します。

CMake を一度呼び出した後に、:option:`-G <cmake -G>` でジェネレータを変更することはできないことに注意して下さい。
ジェネレータを変更するには、まずビルド・ディレクトリを削除して、最初からビルドシステムを生成する必要があります。

Visual Studio のプロジェクトとソリューションのファイルを生成する場合、:manual:`cmake(1)` を初めて実行する時に他のオプションを指定できます。

Visual Studio のツールセットは :option:`cmake -T` オプションで指定できます：

.. code-block:: console

    $ # clang-cl ツールセットでビルドする
    $ cmake.exe .. -G "Visual Studio 16 2019" -A x64 -T ClangCL
    $ # Windows XP をターゲットにしてビルドする
    $ cmake.exe .. -G "Visual Studio 16 2019" -A x64 -T v120_xp

:option:`-A <cmake -A>` オプションでターゲット・アーキテクチャを指定し、:option:`-T <cmake -T>` オプションで使用するツールチェインの詳細を指定します。
たとえば ``-Thost=x64`` はホスト・ツールの 64ビット版を選択するように CMake に指示しています。
次の例は、64ビット版のツールを使い、64ビットのターゲット・アーキテクチャをビルドする方法を示しています：

.. code-block:: console

    $ cmake .. -G "Visual Studio 16 2019" -A x64 -Thost=x64

cmake-gui でジェネレータを選択する
----------------------------------

"Configure" ボタンを押すと、CMake で使用するジェネレータを選択するダイアログが表示されます。

.. image:: GUI-Configure-Dialog.png
   :alt: Configuring a generator

コマンドラインから指定できる全てのジェネレータが :manual:`cmake-gui(1)` でも利用できます。

.. image:: GUI-Choose-Generator.png
   :alt: Choosing a generator

Visual Studio のジェネレータを選択する際は、生成するアーキテクチャを設定するための追加オプションが表示されます。

.. image:: VS-Choose-Arch.png
   :alt: Choosing an architecture for Visual Studio generators

.. _`Setting Build Variables`:

ビルド用の変数をセットする
==========================

ソフトウェアのプロジェクトの中には CMake を呼び出す際にコマンドラインに渡す変数が必要になる場合がよくあります。
以下の表に、もっともよく使用する CMake 変数の一部を示します：

========================================== ============================================================
 変数                                       意味
========================================== ============================================================
 :variable:`CMAKE_PREFIX_PATH`              :guide:`依存するパッケージ <Using Dependencies Guide>` を
                                            探すパス
 :variable:`CMAKE_MODULE_PATH`              CMake の追加モジュールを探すパス
 :variable:`CMAKE_BUILD_TYPE`               デバッグ／最適化のフラグを決定するビルド構成の種類で、
                                            ``Debug`` または ``Release`` 。
                                            これは ``Makefile`` と ``Ninja`` などの単一構成の
                                            ビルドシステムにのみ適用される。 
                                            Visual Studio や Xcode などの複数構成をサポートする
                                            ビルドシステムでは無視する。
 :variable:`CMAKE_INSTALL_PREFIX`           ``install`` のビルド・ターゲットで、ソフトウェアを
                                            インストールするパス
 :variable:`CMAKE_TOOLCHAIN_FILE`           :manual:`ツールチェインと sysroot <cmake-toolchains(7)>`
                                            で説明したクロス・コンパイル用のデータが格納されたファイル
 :variable:`BUILD_SHARED_LIBS`              :command:`add_library` コマンドで値を指定しなかった場合、
                                            静的ライブラリではなく共有ライブラリをビルドする
 :variable:`CMAKE_EXPORT_COMPILE_COMMANDS`  clang 系のツールで使用する ``compile_commands.json`` を
                                            生成する
========================================== ============================================================

プロジェクトのコンポーネントを有効にしたり無効にするなど、プロジェクト固有のビルドを制御する変数が利用できる場合があります。

さまざまなビルドシステムに対応した変数名の付け方を CMake で規定することはありません。ただし接頭子が ``CMAKE_`` の変数は、CMake が提供しているオプションを参照するため、独自の接頭子が必要なサードパーティのプロジェクトでは、これと同名のオプションを定義しないようにして下さい。
:manual:`cmake-gui(1)` ツールは接頭子ごとにグループ化してオプションを表示できるので、サードパーティが定義する独自の接頭子の参照が保証されます。


コマンドラインから変数をセットする
----------------------------------

CMake の変数をコマンドラインから指定できるのは、はじめてビルドシステムを生成する時：

.. code-block:: console

    $ mkdir build
    $ cd build
    $ cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Debug

または、ビルドシステムを生成したあとに :manual:`cmake(1)` を呼び出す時です：

.. code-block:: console

    $ cd build
    $ cmake . -DCMAKE_BUILD_TYPE=Debug

:option:`-U <cmake -U>` オプションは :manual:`cmake(1)` コマンドラインで指定した変数の値を解除する（エントリを削除する）際に使用します：

.. code-block:: console

    $ cd build
    $ cmake . -UMyPackage_DIR

CMake のコマンドラインで生成したビルドシステムは、:manual:`cmake-gui(1)` を使用して変更することができます（その逆も可能）。

:manual:`cmake(1)` ツールで :option:`-C <cmake -C>` オプションを使い初期キャッシュを保存するファイルを指定できます。
これは、同じキャッシュ・エントリを繰り返し必要とするコマンドとスクリプトを単純化する際に便利な機能です。


cmake-gui で変数をセットする
----------------------------

変数は :manual:`cmake-gui(1)` からセットできる場合があります。
"Add Entry" ボタンをクリックすると変数の値をセットするダイアログが表示されます。

.. image:: GUI-Add-Entry.png
   :alt: Editing a cache entry

:manual:`cmake-gui(1)` ユーザ・インタフェースを利用して、既存の変数を編集できます。

CMake キャッシュ
----------------

CMake を実行する時は、使用するコンパイラやその他のツール、そして依存するライブラリなどの在り処を見つける必要があります。
さらにコンパイル時やリンク時のフラグ、あるいは依存するライブラリへのパスは同じものを使い、常に一貫性のあるビルドシステムを再生成できるようにする必要があります。
そのようなパラメータはユーザの環境に固有の情報なので、ユーザによる修正や変更を可能にしておく必要もあります。

CMake を初めて実行すると、ビルド・ディレクトリの中に ``CMakeCache.txt`` というキャッシュ・ファイルが生成されます。このファイルには、そのようなパラメータが Key/Value ペアの形式で格納されています。
ユーザは :manual:`cmake-gui(1)` または :manual:`ccmake(1)` ツールで、このキャッシュ・ファイルの情報を表示したり編集することができます。
これらのツールは、キャシュ情報を編集した後に必要に応じてソフトウェアを再構成したり、ビルドシステムを再生成するための対話型インタフェースを提供しています。
このツールの中では、キャッシュ情報に関連付けられた短い説明文（ヘルプ）も表示される場合があります。

また、それぞれのキャッシュ情報は型を持ち、その型に応じでツールの中で見え方や操作が異なる場合があります。
たとえば ``BOOL`` 型のキャシュ情報はチェックボックスのユーザ・インタフェースで編集でき、``STRING`` 型はテキストボックス、``FILEPATH`` 型は ``STRING`` と同様にファイルシステム上のパスを特定するための手段としてファイル・ダイアログを提供しています。
:manual:`cmake-gui(1)` ツールの場合、``STRING`` 型のキャッシュ情報には、選択可能な文字列が一覧になったドロップ・ダウン形式のリストが提供される場合があります
（詳細はキャッシュ・プロパティの :prop_cache:`STRINGS` を参照して下さい）。

パラメータとして論理型の値については、ソフトウェア・パッケージに同梱されている CMake 関連のファイルの中で :command:`option` コマンドを使って定義できる場合があります。
このコマンドは、パラメータを説明するヘルプとデフォルト値を含むキャッシュ情報を生成します。
通常、このようなキャッシュ情報はソフトウェアに固有の情報であり、ビルド時にテストやサンプルをビルドするかどうか、あるいは例外を有効にしてビルドするかどうか等の設定に影響するものです。


プリセットを使う
================

CMake は通常プリセットで使用する設定を保存するために ``CMakePresets.json`` ファイルの他、ユーザが独自に設定した ``CMakeUserPresets.json`` ファイルを理解します。
これらのプリセットにはビルド・ディレクトリやジェネレータ、キャシュ情報、環境変数、およびその他のコマンドライン・オプションをセットできます。
これらの設定は全てユーザが上書きで再設定できます。
``CMakePresets.json`` ファイルの書式について詳細はマニュアルの :manual:`cmake-presets(7)` を参照して下さい。

コマンドラインからプリセットを使う
----------------------------------

:manual:`cmake(1)` コマンドラインを使う際に :option:`--preset <cmake --preset>` オプションを使ってプリセットを呼び出すことができます。
この :option:`--preset <cmake --preset>` を指定すると、コマンドラインからジェネレータとビルド・ディレクトリの指定は必須ではなくなりますが、指定した場合はこれらプリセットの値が上書きされます。
例えば、次のような ``CMakePresets.json`` ファイルがあるとします：

.. code-block:: json

  {
    "version": 1,
    "configurePresets": [
      {
        "name": "ninja-release",
        "binaryDir": "${sourceDir}/build/${presetName}",
        "generator": "Ninja",
        "cacheVariables": {
          "CMAKE_BUILD_TYPE": "Release"
        }
      }
    ]
  }

そして次のコマンドラインを実行します：

.. code-block:: console

  cmake -S /path/to/source --preset=ninja-release

これにより :generator:`Ninja` というジェネレータを使って ``/path/to/source/build/ninja-release`` ディレクトリの下に、:variable:`CMAKE_BUILD_TYPE` が ``Release`` タイプであるビルド・ディレクトリを生成します。

その一方で、利用可能なプリセットの一覧を表示したい場合は：

.. code-block:: console

  cmake -S /path/to/source --list-presets

このコマンドラインは ``/path/to/source/CMakePresets.json`` と ``/path/to/source/CMakeUsersPresets.json`` の中で利用できるプリセットの一覧を表示します（表示するだけで、ビルド・ディレクトリは生成しません）。

cmake-gui でプリセットを使う
----------------------------

プロジェクトが ``CMakePresets.json`` または ``CMakeUserPresets.json`` ファイルを介してプリセットを利用できる場合、:manual:`cmake-gui(1)` の Source Directory と Binary Directory との間に、プリセットの一覧がドロップ・ダウンメニューの中に表示されます。
その中からプリセットを選択するとバイナリ・ディレクトリ、ジェネレータ、環境変数、そしてキャッシュ情報がセットされますが、これらのすべてはプリセットを選択したあとで上書きで再設定することも可能です。

ビルドシステムを呼び出す
========================

ビルドシステムを生成した後に、特定のビルド・ツールを呼び出してソフトウェアをビルドすることができます。
ジェネレータが IDE の場合、生成されたプロジェクト・ファイルを IDE にロードしてからビルドします。

CMake はビルドに必要な特定のビルド・ツールを理解しているので、コマンドラインからビルドシステムやプロジェクトをビルドする際は、通常は次のコマンドをビルド・ディレクトリの中で実行します：

.. code-block:: console

  $ cmake --build .

オプション :option:`--build <cmake --build>` は :manual:`cmake(1)` の特定の操作モードを有効にします。
これは :manual:`generator <cmake-generators(7)>` に関連付けられた :variable:`CMAKE_MAKE_PROGRAM` コマンド、またはユーザが設定したビルド・ツールを呼び出します。

さらに :option:`--build <cmake --build>` モードでは、ビルドするターゲットを指定する :option:`--target <cmake--build --target>` オプションも指定できます。「ビルドするターゲット」とは、たとえば特定のライブラリや実行形式、または独自のターゲットの他に、``install`` のようなジェネレータに依存したターゲットのことです：

.. code-block:: console

  $ cmake --build . --target myexe

さらに :option:`--build <cmake --build>` モードでは、複数の configuration に対応したジェネレータを使用する場合に、どの configuration を使ってビルドするかを指定するオプション :option:`--config <cmake--build --config>` も指定できます：

.. code-block:: console

  $ cmake --build . --target myexe --config Release

この :option:`--config <cmake--build --config>` オプションは、変数の :variable:`CMAKE_BUILD_TYPE` を指定して :manual:`cmake(1)` を実行した時に選択された configuration でビルドシステムを生成した場合は効果はありません。

一部のビルドシステムでは、ビルド中に呼び出されたコマンドラインの詳細なログが省略される場合があります。
そのような場合は :option:`--verbose <cmake--build --verbose>` オプションを指定できます：

.. code-block:: console

  $ cmake --build . --target myexe --verbose

さらに :option:`--build <cmake --build>` モードでは ``--`` のうしろに特定のコマンドライン・オプションを並べると、ビルド時に呼び出されるビルド・ツールにそれらを渡すことができます。
これは、たとえば CMake が提供していない高レベルなユーザ・インタフェースが必要な場面でビルド・ジョブが失敗しても、そのままビルドを続行するようなオプションを渡したいような場合に便利です。

全てのジェネレータで、CMake を呼び出したあとにビルド・ツールを直接呼び出せます。
たとえば、``make`` は :generator:`Unix Makefiles` というジェネレータで生成したビルドシステムの中で実行できます。
あるいは :generator:`Ninja` というジェネレータであれば ``ninja`` コマンドを実行できます。
通常 IDE が生成したビルドシステムではプロジェクトをビルドするための専用のコマンドライン・ツールを提供しています。


ターゲットを選択する
--------------------

Each executable and library described in the CMake files is a build target, and the buildsystem may describe custom targets, either for internal use, or for user consumption, for example to create documentation.

CMake provides some built-in targets for all buildsystems providing CMake files.

``all``
  The default target used by ``Makefile`` and ``Ninja`` generators.
  Builds all targets in the buildsystem, except those which are excluded by their :prop_tgt:`EXCLUDE_FROM_ALL` target property or :prop_dir:`EXCLUDE_FROM_ALL` directory property.
  The name ``ALL_BUILD`` is used for this purpose for the Xcode and Visual Studio generators.
``help``
  Lists the targets available for build.
  This target is available when using the :generator:`Unix Makefiles` or :generator:`Ninja` generator, and the exact output is tool-specific.
``clean``
  Delete built object files and other output files.
  The ``Makefile`` based generators create a ``clean`` target per directory, so that an individual directory can be cleaned.
  The ``Ninja`` tool provides its own granular ``-t clean`` system.
``test``
  Runs tests.
  This target is only automatically available  if the CMake files provide CTest-based tests.
  See also `テストを実施する`_.
``install``
  Installs the software.
  This target is only automatically available if the software defines install rules with the :command:`install` command.
  See also `ソフトウェアをインストールする`_.
``package``
  Creates a binary package.
  This target is only  automatically available if the CMake files provide CPack-based packages.
``package_source``
  Creates a source package.
  This target is only automatically available if the CMake files provide CPack-based packages.

For ``Makefile`` based systems, ``/fast`` variants of binary build targets are provided.
The ``/fast`` variants are used to build the specified target without regard for its dependencies.
The dependencies are not checked and are not rebuilt if out of date.
The :generator:`Ninja` generator is sufficiently fast at dependency checking that such targets are not provided for that generator.

``Makefile`` based systems also provide build-targets to preprocess, assemble and compile individual files in a particular directory.

.. code-block:: console

  $ make foo.cpp.i
  $ make foo.cpp.s
  $ make foo.cpp.o

The file extension is built into the name of the target because another file with the same name but a different extension may exist.
However, build-targets without the file extension are also provided.

.. code-block:: console

  $ make foo.i
  $ make foo.s
  $ make foo.o

In buildsystems which contain ``foo.c`` and ``foo.cpp``, building the ``foo.i`` target will preprocess both files.

ビルド・ツールを指定する
------------------------

The program invoked by the :option:`--build <cmake --build>`
mode is determined by the :variable:`CMAKE_MAKE_PROGRAM` variable.
For most generators, the particular program does not need to be
configured.

===================== =========================== ===========================
      Generator           Default make program           Alternatives
===================== =========================== ===========================
 XCode                 ``xcodebuild``
 Unix Makefiles        ``make``
 NMake Makefiles       ``nmake``                   ``jom``
 NMake Makefiles JOM   ``jom``                     ``nmake``
 MinGW Makefiles       ``mingw32-make``
 MSYS Makefiles        ``make``
 Ninja                 ``ninja``
 Visual Studio         ``msbuild``
 Watcom WMake          ``wmake``
===================== =========================== ===========================

The ``jom`` tool is capable of reading makefiles of the
``NMake`` flavor and building in parallel, while the
``nmake`` tool always builds serially.  After generating
with the :generator:`NMake Makefiles` generator a user
can run ``jom`` instead of ``nmake``.  The
:option:`--build <cmake --build>`
mode would also use ``jom`` if the
:variable:`CMAKE_MAKE_PROGRAM` was set to ``jom`` while
using the :generator:`NMake Makefiles` generator, and
as a convenience, the :generator:`NMake Makefiles JOM`
generator is provided to find ``jom`` in the normal way
and use it as the :variable:`CMAKE_MAKE_PROGRAM`. For
completeness, ``nmake`` is an alternative tool which
can process the output of the
:generator:`NMake Makefiles JOM` generator, but doing
so would be a pessimization.

ソフトウェアをインストールする
==============================

The :variable:`CMAKE_INSTALL_PREFIX` variable can be
set in the CMake cache to specify where to install the
provided software.  If the provided software has install
rules, specified using the :command:`install` command,
they will install artifacts into that prefix.  On Windows,
the default installation location corresponds to the
``ProgramFiles`` system directory which may be
architecture specific.  On Unix hosts, ``/usr/local`` is
the default installation location.

The :variable:`CMAKE_INSTALL_PREFIX` variable always
refers to the installation prefix on the target
filesystem.

In cross-compiling or packaging scenarios where the
sysroot is read-only or where the sysroot should otherwise
remain pristine, the :variable:`CMAKE_STAGING_PREFIX`
variable can be set to a location to actually install
the files.

The commands:

.. code-block:: console

  $ cmake .. -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DCMAKE_SYSROOT=$HOME/root \
    -DCMAKE_STAGING_PREFIX=/tmp/package
  $ cmake --build .
  $ cmake --build . --target install

result in files being installed to paths such
as ``/tmp/package/lib/libfoo.so`` on the host machine.
The ``/usr/local`` location on the host machine is
not affected.

Some provided software may specify ``uninstall`` rules,
but CMake does not generate such rules by default itself.


テストを実施する
================

The :manual:`ctest(1)` tool is shipped with the CMake
distribution to execute provided tests and report
results.  The ``test`` build-target is provided to run
all available tests, but the :manual:`ctest(1)` tool
allows granular control over which tests to run, how to
run them, and how to report results.  Executing
:manual:`ctest(1)` in the build directory is equivalent
to running the ``test`` target:

.. code-block:: console

  $ ctest

A regular expression can be passed to run only tests
which match the expression.  To run only tests with
``Qt`` in their name:

.. code-block:: console

  $ ctest -R Qt

Tests can be excluded by regular expression too.  To
run only tests without ``Qt`` in their name:

.. code-block:: console

  $ ctest -E Qt

Tests can be run in parallel by passing :option:`-j <ctest -j>`
arguments to :manual:`ctest(1)`:

.. code-block:: console

  $ ctest -R Qt -j8

The environment variable :envvar:`CTEST_PARALLEL_LEVEL`
can alternatively be set to avoid the need to pass
:option:`-j <ctest -j>`.

By default :manual:`ctest(1)` does not print the output
from the tests. The command line argument :option:`-V <ctest -V>`
(or ``--verbose``) enables verbose mode to print the
output from all tests.
The :option:`--output-on-failure <ctest --output-on-failure>`
option prints the test output for failing tests only.
The environment variable :envvar:`CTEST_OUTPUT_ON_FAILURE`
can be set to ``1`` as an alternative to passing the
:option:`--output-on-failure <ctest --output-on-failure>`
option to :manual:`ctest(1)`.
