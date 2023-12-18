依存関係を利用するためのガイド
******************************

.. only:: html

   .. contents::

はじめに
========

一般的に「プロジェクト」は他のプロジェクトやいろいろな目的、そしていろいろな成果物（*Artifacts*）に依存することがよくあります。
CMake は、このような「**依存関係**」をビルドに組み込むためのさまざまな方法を提供します。
これを利用するプロジェクトとユーザは、ニーズにあった最適な方法を柔軟に選択できます。

依存関係をビルドに組み込むために提供されている主な方法は、:command:`find_package` コマンドと :module:`FetchContent` モジュールです。
他に :module:`FindPkgConfig` モジュールを利用できるケースもありますが、これら二つの方法との連携が部分的に欠如しているので、本ガイドでこれ以上は説明しないことにします。

依存関係の中には、独自の「:ref:`依存関係のプロバイダ <dependency_providers>`」なるもので利用可能になるものもあります。
これは、たとえばサードパーティ製のパッケージ・マネージャとか、開発者が実装した独自のコードが該当します。
この依存関係のプロバイダは、上記の二つの方法と連携し、依存関係を組み込む際の柔軟性をさらに拡張します。

.. _prebuilt_find_package:

``find_package()`` で既存パッケージを利用する
=============================================

プロジェクトで必要なパッケージが既にビルドされ、ホスト上のどこかの場所で利用できる場合があります。
そのようなパッケージも CMake でビルドされていたり、または完全に別のビルドシステムが使用されている場合があります。
あるいは、まったくビルドする必要の無いただのファイルの集まりであるかもしれません。
CMake では、このようなケースで利用できる :command:`find_package` コマンドを提供しています。
このコマンドは、プロジェクトまたはユーザから提供された追加情報（ヒント）や追加ディレクトリと連動して、システムで既知の場所からファイルを探し出します。
さらに、ファイルだけではなくパッケージに含まれるコンポーネントやオプションのパッケージもサポートしています。
パッケージや特定のファイルが見つかったかどうかに応じて、プロジェクト内で独自に対応できるように、その結果を格納するための変数が自動的に提供されます。

プロジェクトで、このコマンドを使う場合は「:ref:`basic signature`」に従う必要があります。
ほとんどの場合、引数としてパッケージの名前や、場合によってはパッケージのバージョン、そして必須の依存関係ならば ``REQUIRED`` というキーワードを渡すことになります。
また、パッケージのコンポーネント一式を指定するケースもあるかもしれません。

.. code-block:: cmake
  :caption: ``find_package()`` コマンドの呼び出し例

  find_package(Catch2)
  find_package(GTest REQUIRED)
  find_package(Boost 1.79 COMPONENTS date_time)

この :command:`find_package` コマンドは検索モードを二つ提供しています：

**Config モード**
  このモードを使うと、通常はパッケージが配布しインストールしたファイルを探します。
  これらのファイルの詳細は常にパッケージと同期されているはずなので、他のモードよりも信頼性の高い結果が得られます。

**Module モード**
  全てのパッケージが CMake に対応しているわけではありません。
  多くパッケージが **Config モード** を機能させるのに必要なファイルを提供していません。
  そのような場合は、プロジェクトまたは CMake で、パッケージ向けの「:ref:`Find モジュール <Libraries not Providing Config-file Packages>`」なるものを用意できます。
  通常 Find モジュールは、そのパッケージが何のファイルを配布し、それらをどのようにしてプロジェクト側に提供しているのかを解決する実装を持っています。
  また通常 Find モジュールはパッケージと別々に提供されているため、検索結果の信頼性はそれほど高くありません。
  つまり、パッケージと Find モジュールは個別に保守され、異なるスケジュールに従ってリリースされる場合が多いので、簡単に情報が古くなってしまう可能性があります。

この :command:`find_package` コマンドは、受け取ったオプションに応じて、二つあるモードの一方を使うか、あるいは両方を使うかを決めます。
呼び出せるコマンドの「:ref:`basic signature`」を制限することで、両方のモードを使った依存関係の解決が可能になります。
つまり、いろいろなオプションを渡してしまうと、2つあるモードのうち1つだけに使用が制限されてしまい、このコマンドの本来の能力を発揮できないかもしれません。
この複雑なトピックについて詳細は、この :command:`find_package` のドキュメントを参照してみて下さい。

いずれのモードでも、ユーザは :manual:`cmake(1)` のコマンドライン、または :manual:`ccmake(1)` や :manual:`cmake-gui(1)` などの GUI ツールでキャッシュ変数を指定することで、パッケージを探す場所をカスタマイズすることができます。
この指定方法については「:ref:`ユーザ操作ガイド <Setting Build Variables>`」を参照して下さい。

.. _Libraries providing Config-file packages:

パッケージの Config ファイル
----------------------------

CMake で使用される実行形式やライブラリ、ヘッダ、その他のファイルを提供する方法として、サードパーティに推奨されているのが「:ref:`Config ファイル <Config File Packages>`」です。
このファイルはパッケージに同梱されているテキスト・ファイルで、CMake でビルドするターゲットや CMake で参照できる変数、そして CMake コマンドなどを定義します。
Config ファイルは普通の CMake スクリプトで、:command:`find_package` コマンドによって読み込まれます。

通常 Config ファイルは ``lib/cmake/<PackageName>`` に従ったディレクトリの中にありますが、別のディレクトリにある場合もあります（「:ref:`search procedure`」も参照して下さい）。
ここで ``<PackageName>`` は  :command:`find_package` コマンドの先頭オプションとして渡したパッケージの名前です。
あるいは ``NAMES`` オプションで、代替えの名前を指定できます：

.. code-block:: cmake
  :caption: パッケージを探す際にその別名を渡す例

  find_package(SomeThing
    NAMES
      SameThingOtherName   # パッケージのもう一つの名前
      SomeThing            # 正規のパッケージ名でも探す
  )

Config ファイルの名前は ``<PackageName>Config.cmake`` または ``<LowercasePackageName>-config.cmake`` にして下さい（本ガイドでは前者を使用していますが、両方ともサポートしています）。
このファイルは CMake でのパッケージ検索のエントリ・ポイントになります。
オプションとして、``<PackageName>ConfigVersion.cmake`` または ``<LowercasePackageName>-config-version.cmake`` という名前のファイルも同じディレクトリにおいて下さい。
このファイルは、:command:`find_package` コマンドの呼び出しでパッケージのバージョンが指定された際に、バージョンによる制約を満足しているかどうかをチェックする際に使用します。
なお、``<PackageName>ConfigVersion.cmake``  が存在していても、:command:`find_package` コマンドにオプションとして任意のバージョンを渡すことができます。

検索したいパッケージの ``<PackageName>Config.cmake`` ファイルが存在し、バージョンの制約を満足している場合、:command:`find_package` コマンドはそのパッケージが完全な形で提供されているとみなします。

場合によっては、プロジェクトで利用できる CMake コマンドや「:ref:`imported targets`」を提供する追加のファイルが存在している場合があります。
CMake は、このようなファイルには命名規則を強制していません。
:command:`include` コマンドを使うと、このようなファイルはメインの ``<PackageName>Config.cmake`` ファイルに関連づけされます。
通常、このようなファイルはメインの ``<PackageName>Config.cmake`` ファイルが自動的に取り込むので、 :command:`find_package` コマンドを呼び出す他に追加の作業はありません。

もし :ref:`CMake に既知のディレクトリ下 <search procedure>` にパッケージがあれば、:command:`find_package` コマンドの呼び出しは成功します。
CMake に認識される場所は、ホストのプラットフォーム固有のディレクトリやフォルダです。
たとえば、Linux 系のプラットフォーム標準のパッケージ・マネージャを使ってインストールされたパッケージならば、自動的に ``/usr`` を Prefix としたディレクトリ下にあります。
Windows 系のプラットフォームで ``Program Files`` フォルダにインストールされているパッケージも同様に自動的に見つけます。

``/opt/mylib`` とか ``$HOME/dev/prefix`` などのような CMake が認識していない場所にパッケージがある場合、「なんらかのヒント」無しで自動的にパッケージを見つけることはできません。
そのため CMake はパッケージを見つける場所を指定する方法をいくつか提供しています。

CMake 変数の :variable:`CMAKE_PREFIX_PATH` は :ref:`CMake を呼び出すとセット <Setting Build Variables>` されます。
この変数の値は :ref:`Config ファイル <Config File Packages>` を探すためのベース・ディレクトリを要素とする :ref:`リスト <CMake Language Lists>` として扱います。
たとえば ``/opt/somepackage`` 下にインストールされたパッケージは、``/opt/somepackage/lib/cmake/somePackage/SomePackageConfig.cmake`` という Config ファイルをインストールします。
その場合 Prefix の一つとして ``/opt/somepackage`` を :variable:`CMAKE_PREFIX_PATH` に追加しておく必要があります。

この ``CMAKE_PREFIX_PATH`` にはパッケージを探す際に参照する Prefix をセットします。
環境変数の ``PATH`` と同様に、これは :ref:`リスト <CMake Language Lists>` ですが、ホストのプラットフォーム固有のディレクトリ区切り文字（Windows 系プラットフォームの場合は ``;``、UNIX 系プラットフォームの場合は ``:``）を使って下さい。

複数の Prefix 配下を探したい場合とか、複数のパッケージが同じ Prefix 配下にインストールされているような場合に、この :variable:`CMAKE_PREFIX_PATH` は便利です。
パッケージを指すパスも ``<PackageName>_DIR`` に従った変数（たとえば ``SomePackage_DIR``）をセットすることで指定できます。
ただし、この変数には Prefix ではなく、Config ファイルの類を配置したディレクトリへの絶対パス（先の例だと ``/opt/somepackage/lib/cmake/SomePackage``）をセットする点が違うので注意して下さい。
パッケージやファイルの検索に影響を与えそうな、その他の CMake 変数や環境変数については、:command:`find_package` コマンドのドキュメントを参照して下さい。

.. _Libraries not Providing Config-file Packages:

パッケージの Find モジュール
----------------------------

:ref:`Config ファイル <Libraries providing Config-file packages>` を提供していないパッケージであっても、Find モジュールのファイル（``Find<PackageName>.cmake``）が利用できれば、依然として :command:`find_package` コマンドで見つけることができます。
Find モジュールと Config ファイルの違いは次のとおりです：

#. Find モジュールのファイルはパッケージ自身で提供すべきものではない。
#. Find モジュールのファイル（``Find<PackageName>.cmake``）が利用できることと、そのパッケージ（の一部）が利用できることは同義ではない。
#. CMake は Find モジュールのファイル（``Find<PackageName>.cmake``）の :variable:`CMAKE_PREFIX_PATH` にリストされた場所は検索しない。
   代わりに CMake 変数の :variable:`CMAKE_MODULE_PATH` にセットされている場所で Find モジュールのファイルを探す。
   これら CMake 変数の一般的な使い方は：

   - ユーザが CMake の実行時に :variable:`CMAKE_MODULE_PATH` をセットする。
   - CMake のプロジェクトがローカルの Find モジュールの利用を許可するために、そのファイルがある場所を :variable:`CMAKE_MODULE_PATH` に追加する。

#. CMake には、一部の :manual:`サードパーティ製のパッケージ <cmake-modules(7)>` に対する Find モジュールのファイル（``Find<PackageName>.cmake``）を同梱している。
   ただ CMake にとって、これらのファイルの保守は負担であり、このファイルが実パッケージの最新版から遅れた対応になってしまうことは珍しいことではない。
   一般に、新しい Find モジュールのファイルが CMake に同梱されることが無くなっている。
   あなたのプロジェクトで、可能であれば、 パッケージの Config ファイルを提供するよう上流のプロジェクトに働きかける必要がある。
   さもなくば、あなたのプロジェクトでサードパーティ製パッケージに対応する Find モジュールを独自に提供し続けていく必要がある。

Find モジュールのファイル作成方法について詳細は「:ref:`Find Modules`」を参照して下さい。

.. _Imported Targets from Packages:

IMPORTED なターゲット
---------------------

:ref:`Config ファイル <Libraries providing Config-file packages>` と :ref:`Find モジュールのファイル <Libraries not Providing Config-file Packages>` の両方で「:ref:`Imported targets`」を定義できます。
通常、このターゲット名は ``SomePrefix::ThingName`` の形式に従います。
このようなターゲットが利用できる場合、プロジェクトは、同様に定義されてる CMake 変数の代わりに、こちらのターゲットを使用することを優先しなければなりません。
通常、このようなターゲットは「:ref:`利用要件 <Target Usage Requirements>`」（*Usage Requirements*）を持ち、ヘッダファイルの検索パスやコンパイラの定義などを、それらをリンクする他のターゲット（たとえば :command:`target_link_libraries`）に自動的に適用されます。
これは、手動で同じように適用することよりも安全で便利です。
必要であれば、パッケージや Find モジュールのドキュメントを参照して :ref:`Imported targets` を確認してみて下さい。

また :ref:`Imported targets` はビルドシステム固有のいろいろなパスもカプセル化しています。
つまり、このターゲットにはバイナリ（ライブラリや実行形式）のインストール先やコンパイラのフラグ、そしてその他ビルドシステムに依存した情報が含まれています。
ただし Find モジュールは、Config ファイルよりも信頼性の低い情報を提供する場合があります。

たとえばサードパーティ製のパッケージを検索し、そこからライブラリを使用するコードは次のようになります：

.. code-block:: cmake

  cmake_minimum_required(VERSION 3.10)
  project(MyExeProject VERSION 1.0.0)

  # このプロジェクトが提供しているローカルの Find モジュールを利用する
  list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

  find_package(SomePackage REQUIRED)
  add_executable(MyExe main.cpp)
  target_link_libraries(MyExe PRIVATE SomePrefix::LibName)

この :command:`find_package` コマンドの呼び出しは Config ファイルまたは Find モジュールによって解決される点に注意して下さい。
この呼び出しは :ref:`basic signature` でサポートしているオプションだけ使います。
たとえば ``${CMAKE_CURRENT_SOURCE_DIR}/cmake`` にある Find モジュールのファイル（``FindSomePackage.cmake``）を使うと、:command:`find_package` コマンドは Module モードで成功します。
この Find モジュールのファイルが存在しない場合、CMake は Config ファイルを探します。


``FetchContent`` モジュールでソースから依存関係を生成する
=========================================================

CMake で依存関係を利用するために、必ずしも既存のパッケージが必要であるという訳ではありません。
依存関係は、プロジェクトの一部としてソースからも生成できます。
:module:`FetchContent` モジュールはコンテンツ（通常はソース・ファイルですが、何でも構いません）をダウンロードし、それをプロジェクトに追加する機能を提供しています。
これにより、追加されたコンテンツは、あたかもプロジェクトのソースの一部であるかのように、他のソースと共にビルドされます。

一般的なケースは、プロジェクトはまず利用する全ての依存関係を抽出し、それらを利用できるようにするために CMake に要求します。

次は、この原理を例として示したものです（詳細については :ref:`fetch-content-examples` を参照して下さい）：

.. code-block:: cmake

  include(FetchContent)
  FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        703bd9caab50b139428cea1aaff9974ebee5742e # release-1.10.0
  )
  FetchContent_Declare(
    Catch2
    GIT_REPOSITORY https://github.com/catchorg/Catch2.git
    GIT_TAG        605a34765aa5d5ecbf476b4598a862ada971b0cc # v3.0.1
  )
  FetchContent_MakeAvailable(googletest Catch2)

この :module:`FetchContent` モジュールは、任意の URL からアーカイブ（さまざまなアーカイブの形式をサポートしています）をダウンロードして展開する機能の他に、Git、Subversion、そして Mercurial などたくさんの種類のリポジトリを含め、さまざまなダウンロード機能をサポートしています。
さらに、任意のユースケースに対応できるようにするために、独自のダウンロードやアップデート、そして ``patch`` コマンドも利用できます。

このモジュールを使ってプロジェクトに依存関係を追加すると、プロジェクトはその他のターゲットと同様に、その依存関係をターゲットにリンクします。
もし依存関係が ``SomePrefix::ThingName`` に従った名前空間付きのターゲットを提供している場合、名前空間のあるターゲットにだけリンクする必要があります。
これが推奨されている理由については、次のセクションを参照して下さい。

なお、この方法で全ての依存関係をプロジェクトに組み込めるわけではありません。
依存関係の定義の中には、プロジェクトまたは依存関係のターゲットと名前が衝突するターゲットを定義しているものがあります。
:command:`add_executable` と :command:`add_library` コマンドで生成した実行形式やライブラリのターゲットはグローバルな名前空間を持つため、それぞれがビルド全体で一意な名前でなければなりません。
もし任意の依存関係がそのようなターゲットを追加した場合、この方法で依存関係をプロジェクトに組み込めなくなります。

``find_package()`` コマンドと ``FetchContent`` モジュールの両方を使う
=====================================================================

.. versionadded:: 3.24

CMake プロジェクトは :command:`find_package` コマンド、または :module:`FetchContent` モジュールによる依存関係の追加をサポートしています。
このような依存関係は、パッケージをインストールした場合とソースからビルドした場合の双方のケースで、同じ名前空間を持つターゲットが定義されることを保証する必要があります。
これにより、この依存関係を使用するプロジェクトは名前空間付きのターゲットにリンクし、双方のケースでシームレスに処理できます。

プロジェクトは、:command:`FetchContent_Declare` コマンドの ``FIND_PACKAGE_ARGS`` オプションを使用して、:command:`find_package` コマンドか :module:`FetchContent` モジュールかのどちらの方法で依存関係を受け入れたのかを宣言することができます。
:command:`FetchContent_MakeAvailable` コマンドは、この ``FIND_PACKAGE_ARGS`` の引数を利用し、まず最初に :command:`find_package` コマンドを呼び出して依存関係の解決を試みます。
依存関係を解決できなかった場合は、（前述のとおり）ソースから依存関係を生成します。

.. code-block:: cmake

  include(FetchContent)
  FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        703bd9caab50b139428cea1aaff9974ebee5742e # release-1.10.0
    FIND_PACKAGE_ARGS NAMES GTest
  )
  FetchContent_MakeAvailable(googletest)

  add_executable(ThingUnitTest thing_ut.cpp)
  target_link_libraries(ThingUnitTest GTest::gtest_main)

この例は、まず :command:`find_package(googletest NAMES GTest) <find_package>` コマンドを呼び出します。
CMake は :module:`FindGTest` というモジュールを提供しているので、どこかにインストールされた GTest パッケージを見つけると、これを有効化するので、ソースから依存関係は生成されません。
対して GTest パッケージが見つからなかったら、依存関係を「ソースから」生成します。
いずれの場合でも、``GTest::gtest_main`` というターゲットが定義されているものと期待されるため、ユニットテストの実行形式をそのターゲットにリンクすることになります。

また、この操作は :variable:`FETCHCONTENT_TRY_FIND_PACKAGE_MODE` 変数を使って利用できます。
この変数に ``NEVER`` をセットすると :command:`find_package` コマンドの呼び出しは行いません。
この変数に ``ALWAYS`` をセットすると、``FIND_PACKAGE_ARGS`` オプションを指定しなくても :command:`find_package` コマンドの呼び出しを試みます（この ``ALWAYS`` は注意して使う必要があります）。

またはプロジェクトは、特定の依存関係をソースから生成する必要があると判断する場合があります。
これは、依存関係のパッチの適用が必要か、または未だリリースしていないバージョンが必要か、あるいは全ての依存関係をソースから生成する必要かというポリシーを解決する場合に使われる方法です。
プロジェクトでは、:command:`FetchContent_Declare` コマンドに ``OVERRIDE_FIND_PACKAGE`` オプションを渡すことで、この判断を強制できます。
その際は、依存関係を解決するために :command:`find_package` コマンドを呼び出すと :command:`FetchContent_MakeAvailable` コマンドにリダイレクトされます。

.. code-block:: cmake

  include(FetchContent)
  FetchContent_Declare(
    Catch2
    URL https://intranet.mycomp.com/vendored/Catch2_2.13.4_patched.tgz
    URL_HASH MD5=abc123...
    OVERRIDE_FIND_PACKAGE
  )

  # 次の呼び出しは自動的に FetchContent_MakeAvailable(Catch2) の呼び出しにリダイレクトされる
  find_package(Catch2)

さらに高度な使い方については CMake 変数の :variable:`CMAKE_FIND_PACKAGE_REDIRECTS_DIR` を参照して下さい。

.. _dependency_providers_overview:

依存関係のプロバイダ
====================

.. versionadded:: 3.24

前のセクションでは、プロジェクトがその依存関係を指定するために使う方法について説明しました。
プロジェクトは、期待されているもの（多くの場合はインポートしたターゲットの一部）を提供する限り、依存関係がどこから要求されているのかあまり気にする必要が無いことが理想です。
.. The project says what it needs and may also specify where to get it from, in the absence of any other details, so that it can still be built out-of-the-box.（FIXME: 意味不明）


一方、開発者の場合は、プロジェクトに「依存関係を提供する方法」について興味があるかもしれません。
たとえば、自分がビルドしたパッケージの特定のバージョンを使用したいとか、サードパーティのパッケージ・マネージャを使用したいとか、セキュリティやパフォーマンス上の理由から一部のビルド・リクエストをビルドシステムの別の URL にリダイレクトしたいとか。
CMake は、このようなケースを「:ref:`dependency_providers`」でサポートしています。

A dependency provider can be set to intercept :command:`find_package` and :command:`FetchContent_MakeAvailable` calls.
The provider is given an opportunity to satisfy such requests before falling back to the built-in implementation if the provider doesn't fulfill it.

Only one dependency provider can be set, and it can only be set at a very specific point early in the CMake run.
The :variable:`CMAKE_PROJECT_TOP_LEVEL_INCLUDES` variable lists CMake files that will be read while processing the first :command:`project()` call (and only that call).
This is the only time a dependency provider may be set.
At most, one single provider is expected to be used throughout the whole project.

For some scenarios, the user wouldn't need to know the details of how the dependency provider is set.
A third party may provide a file that can be added to :variable:`CMAKE_PROJECT_TOP_LEVEL_INCLUDES`, which will set up the dependency provider on the user's behalf.
This is the recommended approach for package managers.
The developer can use such a file like so::

  cmake -DCMAKE_PROJECT_TOP_LEVEL_INCLUDES=/path/to/package_manager/setup.cmake ...

For details on how to implement your own custom dependency provider, see the :command:`cmake_language(SET_DEPENDENCY_PROVIDER)` command.
