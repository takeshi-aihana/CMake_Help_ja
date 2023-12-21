find_package
------------

.. |FIND_XXX| replace:: ``find_package``
.. |FIND_ARGS_XXX| replace:: <PackageName>
.. |FIND_XXX_REGISTRY_VIEW_DEFAULT| replace:: ``TARGET``
.. |CMAKE_FIND_ROOT_PATH_MODE_XXX| replace::
   :variable:`CMAKE_FIND_ROOT_PATH_MODE_PACKAGE`

.. only:: html

   .. contents::

.. note::
  「:guide:`依存関係を利用するためのガイド`」では、このコマンドの一般的なトピックについて、その概要を説明しています。
  このガイドは、:module:`FetchContent` モジュールとの関係を含め、``find_package()`` コマンドが CMake の全体像のどこに当てはまるのかについて、より広範な概要を提供しています。
  本ドキュメントの前に、こちらのガイドを事前に目を通しておくことをお勧めします。

パッケージ（通常はプロジェクト外から提供されるもの）を見つけ出し、そのパッケージ固有の詳細をプロジェクトに取り込む。
このコマンドの呼び出しは「:ref:`依存関係のプロバイダ <dependency_providers>`」によって割り込まれる場合がある。

検索処理のモデル
^^^^^^^^^^^^^^^^

このコマンドにはパッケージを検索するモードがいくつかあります：

**Module モード**
  このモードでは、CMake は ``Find<PackageName>.cmake`` というファイルを探します。
  その際は、まず最初に CMake 変数の :variable:`CMAKE_MODULE_PATH` にセットされている場所を探し、それから CMake がインストールした :ref:`Find モジュール <Find Modules>` を検索します。
  もし ``Find<PackageName>.cmake`` ファイルが見つかったら、そのファイルを読み込んで処理します。
  その際はパッケージを探して、そのバージョンをチェックし、必要なメッセージを出力します。
  一部の Find モジュールはバージョン管理するものがあります（それぞれの Find モジュールのドキュメントを確認してみて下さい）。

  通常、この ``Find<PackageName>.cmake`` ファイルはパッケージからは提供されません。
  むしろ OS のディストリビュータ、CMake、あるいは  ``find_package()`` コマンドを呼び出すプロジェクトなど、検索するパッケージ以外の何らかによって提供されるものです。
  一方、外部から提供される :ref:`Find モジュール <Find Modules>` は「放置される」傾向にあり、その内容は古くなりやすいです。
  通常、このモードでは特定のライブラリ、特定のファイル、そしてパッケージの類を探します。

  このモードは「:ref:`basic signature`」でしか呼び出せません。

**Config モード**
  このモードでは、CMake は ``<lowercasePackageName>-config.cmake`` または ``<PackageName>Config.cmake`` というファイルを探します。
  また、バージョンの詳細が指定された場合は ``<lowercasePackageName>-config-version.cmake`` または ``<PackageName>ConfigVersion.cmake`` という Version ファイルも探します（これらのファイルの使い方については「:ref:`version selection`」を参照して下さい）。

  この Config モードでは、検索するパッケージ名を :ref:`リスト <CMake Language Lists>` として指定できます。
  CMake が Config ファイルと Version ファイルを探す場所は Module モードの検索よりも複雑です。詳細は「:ref:`search procedure`」を参照して下さい。

  通常 Config ファイルと Version ファイルは任意のパッケージの一部としてインストールされるので、Find モジュールよりも信頼性が高い傾向があります。
  また、これらのファイルにはパッケージの直接的な情報が含まれています。

  このモードは「:ref:`basic signature`」と「:ref:`full signature`」 の両方から呼び出せます。

**FetchContent モジュールへの転送モード**
  .. versionadded:: 3.24
    この ``find_package()`` コマンドの呼び出しを :module:`FetchContent` モジュールが提供しているパッケージに転送できます。
    コマンドを呼び出した側には、パッケージの検索ロジックをバイパスし、パッケージの情報を利用しないことを除いて、Config モードと同じ動きになります。
    詳細は :command:`FetchContent_Declare` コマンドと :command:`FetchContent_MakeAvailable` コマンドを参照して下さい。

:module:`FetchContent` モジュールが提供した任意のパッケージに ``find_package()`` コマンドの呼び出しを転送しない場合、このコマンドの引数で Module モードと Config モードのどちらを使うかがを決まります。
「:ref:`basic signature`」を使うと、このコマンドは最初に Module モードで動きます。
Module モードでパッケージが見つからなかったら、次は Config モードで探します。
なお、CMake 変数の :variable:`CMAKE_FIND_PACKAGE_PREFER_CONFIG` でこのモードの順番を逆転させ、Config モードでパッケージが見つからなかったら Module モードで検索させることも可能です。
また「:ref:`basic signature`」の場合は ``MODUELE`` オプションで Module オードだけ使うよう強制させることもできます。
対して「:ref:`full signature`」は Config モードしか選択できません。

できるだけ通常は「:ref:`basic signature`」を使ってパッケージを探すようにして下さい。そうすることで、最終的にはどちらかのモードでパッケージが見つかるからです。
プロジェクトで Config ファイルを提供することを検討しているプロジェクト管理者は、「:ref:`full signature`」とそれに続くセクションで説明されているとおり、このコマンドの全体像を理解しておく必要があります。

.. _`basic signature`:

コマンドの基本形
^^^^^^^^^^^^^^^^

.. parsed-literal::

  find_package(<PackageName> [version] [EXACT] [QUIET] [MODULE]
               [REQUIRED] [[COMPONENTS] [components...]]
               [OPTIONAL_COMPONENTS components...]
               [REGISTRY_VIEW  (64|32|64_32|32_64|HOST|TARGET|BOTH)]
               [GLOBAL]
               [NO_POLICY_SCOPE]
               [BYPASS_PROVIDER])

「コマンドの基本形」は Module モードと Config モードの両モードをサポートしているコマンド呼び出しのシグネチャです。
``MODULE`` オプションは、パッケージを探す際に Module モードだけ使用でき、パッケージが見つからなくても Config モードでは探しません。

どちらのモードでも、パッケージが見つかったかどうかを示す ``<PackageName>_FOUND`` 変数がセットされます。
見つかったパッケージ固有の情報は別の変数や :ref:`Imported Targets` を介して取得できます。
``QUIET`` オプションはメッセージの出力を無効にします（ただし ``REQUIRED`` オプションを指定した場合を除く）。
そして ``REQUIRED`` オプションは、パッケージが見つからなかったらエラー・メッセージを出力してコマンドの処理を停止します。

``COMPONENTS`` オプションには、必要なパッケージのコンポーネント（ファイル）名を要素とする :ref:`リスト <CMake Language Lists>` を指定します。
このリストに挙げたコンポーネントのいずれかが見つからなかったら、パッケージが見つからなかったものとみなします。
その際に ``REQUIRED`` オプションも指定していた場合は致命的なエラーとして扱われます。
なお省略形として、 ``REQUIRED`` オプションを指定する際は ``COMPONENTS`` オプションを省略し、``REQUIRED`` の後ろにそのままコンポーネントのリストを指定できます。

オプション扱いのコンポーネントのリストは ``OPTIONAL_COMPONENTS`` オプションに指定します。
たとえこのオプションに追加したコンポーネントのいずれかが見つからなくても、``COMPONENTS`` に指定したコンポーネントが全て見つかっていれば、そのパッケージが見つかったものとみなします。

CMake は、ここで見つかったコンポーネントは全てターゲットのパッケージが定義したものであることを期待します。
パッケージに与えられたコンポーネントの情報をどのように解釈するかは、そのパッケージ次第ですが、この期待に従う必要があります。
ここで、コンポーネントを指定せずにこのコマンドを呼び出すと（期待する）検索処理は行われません。
つまりパッケージは「全てのコンポーネントを見つける」、「全てのコンポーネントを見つけない」、「利用可能なコンポーネントを検索する」といった状況を想定しておく必要があります。

.. versionadded:: 3.24
  ``REGISTRY_VIEW`` オプションには、どのレジストリ・ビューを照会するかを指定します。
  このオプションはホストのプラットフォームが Windows の場合にのみ意味を持ち、その他のプラットフォームでは無視されます。
  形式的には、パッケージに与えられたレジストリ・ビューの情報をどのように解釈するかはパッケージ次第です。

.. versionadded:: 3.24
  ``GLOBAL`` オプションを指定すると、:ref:`Imported Targets` が全てグローバルなスコープに昇格します。
  もしくは CMake 変数の :variable:`CMAKE_FIND_PACKAGE_TARGETS_GLOBAL` で、この機能を有効にできます。

.. _FIND_PACKAGE_VERSION_FORMAT:

``[version]`` オプションで、見つかったパッケージと互換性があるバージョンを要求します。
ここで指定できるバージョンの書式は2つあります：

  * バージョンを単一で指定する場合は ``major[.minor[.patch[.tweak]]]`` （各アイテムは数値）
  * バージョンを任意の範囲で指定する場合は ``versionMin...[<]versionMax`` （``versionMin`` と ``versionMax`` はそれぞれ「バージョンを単一で指定する」場合と同じ書式）
    デフォルトは双方 ``versionMin`` および ``versionMax`` のバージョンを含んだ範囲になる。
    ここで ``<`` を指定すると ``versionMax`` は含めない範囲になる。
    このバージョン範囲は CMake のバージョン 3.19 以降でのサポートになる。

``EXACT`` オプションは、完全に一致するバージョンを要求します。
このオプションは「バージョンを任意の範囲で指定する」場合と一緒に指定できません（指定しても無視されます）。

このコマンドを ``[version]`` や ``COMPONENTS`` を指定せずに「:ref:`Find Modules`」で再起呼び出しした時は、このコマンドをプロジェクト（外部）から呼び出した時のオプションが自動的に転送されます（``[version]`` の ``EXACT`` も同様 ）。
このバージョン制約の機能は、現在はパッケージ単位でのみのサポートです（この下の「`Version Selection`_」項を参照のこと ）。
つまりバージョンの範囲が指定されても、パッケージ自身が単一のバージョンを想定した設計になっている場合、バージョン範囲の ``versionMax`` を無視して、バージョン範囲の ``versionMin`` だけを受け入れるようになっています。

``NO_POLICY_SCOPE`` オプションの詳細については、:command:`cmake_policy`  コマンドのドキュメントを参照して下さい。

.. versionadded:: 3.24
  ``BYPASS_PROVIDER`` オプションは、「:ref:`依存関係のプロバイダ <dependency_providers>`」が ``find_package()`` コマンドを呼び出した時にだけ指定できます。
  このプロバイダは CMake 内の ``find_package()`` の実装を直接呼び出し、``find_package()`` の実装で呼び出されることがないようにします。
  CMake の将来のバージョンでは、このオプションをプロバイダ以外から使おうとすると、致命的なエラーで停止します。

.. _`full signature`:

コマンドの完全形
^^^^^^^^^^^^^^^^

.. parsed-literal::

  find_package(<PackageName> [version] [EXACT] [QUIET]
               [REQUIRED] [[COMPONENTS] [components...]]
               [OPTIONAL_COMPONENTS components...]
               [CONFIG|NO_MODULE]
               [GLOBAL]
               [NO_POLICY_SCOPE]
               [BYPASS_PROVIDER]
               [NAMES name1 [name2 ...]]
               [CONFIGS config1 [config2 ...]]
               [HINTS path1 [path2 ... ]]
               [PATHS path1 [path2 ... ]]
               [REGISTRY_VIEW  (64|32|64_32|32_64|HOST|TARGET|BOTH)]
               [PATH_SUFFIXES suffix1 [suffix2 ...]]
               [NO_DEFAULT_PATH]
               [NO_PACKAGE_ROOT_PATH]
               [NO_CMAKE_PATH]
               [NO_CMAKE_ENVIRONMENT_PATH]
               [NO_SYSTEM_ENVIRONMENT_PATH]
               [NO_CMAKE_PACKAGE_REGISTRY]
               [NO_CMAKE_BUILDS_PATH] # Deprecated; does nothing.
               [NO_CMAKE_SYSTEM_PATH]
               [NO_CMAKE_INSTALL_PREFIX]
               [NO_CMAKE_SYSTEM_PACKAGE_REGISTRY]
               [CMAKE_FIND_ROOT_PATH_BOTH |
                ONLY_CMAKE_FIND_ROOT_PATH |
                NO_CMAKE_FIND_ROOT_PATH])

``CONFIG`` オプション、またはこれと同じ意味を持つ ``NO_MODULE`` オプション、あるいは 「:ref:`basic signature`」に無いオプションを指定すると、すべて強制的に Config モードによる検索を行います。
この Config モードは、Module モードを使った検索はスキップします。

Config モードによる検索では、パッケージが提供している「:ref:`Config ファイル <Config File Packages>`」を探します。
CMake は、見つかった Config ファイルの場所（ディレクトリ）を格納する ``<PackageName>_DIR`` という CMake 変数を自動的に生成します。
このコマンドは、デフォルトで ``<PackageName>`` という名前のパッケージを検索の対象にします。
なお ``NAMES`` オプションを指定すると、このオプションに渡した名前を ``<PackageName>`` の代わりに使います。
この名前は、コマンドの呼び出しを :module:`FetchContent` モジュールが提供したパッケージに転送するかを決める際にも使われます。

このコマンドは、指定した ``<PackageName>`` （パッケージ名）ごとに ``<PackageName>Config.cmake`` または ``<lowercasePackageName>-config.cmake`` 形式の Config ファイルを探します。
``CONFIGS`` オプションを使って、この Config ファイルの形式を置き換えることができます。
検索の手順については「:ref:`search procedure`」を参照して下さい。
Config ファイルが見つかったら、まず「:ref:`version constraint <version selection>`」をチェックし、それを満足したら Config ファイルを読み込みます。
Config ファイルはパッケージによって提供されるので、このファイルにはパッケージに含まれているコンポーネントとその格納場所が記載されています。
見つかった Config ファイルの絶対パスは CMake変数の ``<PackageName>_CONFIG`` に格納されます。

適切なバージョンを持つパッケージの検索中に、CMake がチェックする全ての Config ファイルは CMake変数の ``<PackageName>_CONSIDERED_CONFIGS`` に、そしてパッケージのバージョンは ``<PackageName>_CONSIDERED_VERSIONS`` 変数にそれぞれ格納されます。

パッケージの Config ファイルが見つからなかったら、CMake はエラーを発行します（ただし ``QUIET`` オプションを指定した場合は除く）。
``REQUIRED`` オプションを指定して、パッケージが見つからなかったら CMake は致命的なエラーを発行し、このコマンドの処理は停止します。
``<PackageName>_DIR`` に格納された Config ファイルが存在しなかったら CMake はそれを無視し、最初に戻って次のパッケージの検索を行います。

Config ファイルを提供するパッケージを保守する人は、「:ref:`search procedure`」で説明している追加のオプションを使わずに、Config ファイルを見つけられるようにファイルに適切な名前を付けてインストールされるようにして下さい。

.. _`search procedure`:

Config モードの検索詳細
^^^^^^^^^^^^^^^^^^^^^^^

.. note::
  Config モードで実行時、「:ref:`basic signature`」や「:ref:`full signature`」の呼び出し方に関係なく、ここで説明している手順が適用されます。

.. versionadded:: 3.24
  Module モードも含め、``find_package()`` コマンドの呼び出しは全て、まず CMake 変数の :variable:`CMAKE_FIND_PACKAGE_REDIRECTS_DIR` が指すディレクトリから Config ファイルの検索を行います。
  :module:`FetchContent` モジュール、または CMake 自身が、このディレクトリにファイルを書き込んで ``find_package()`` コマンドの呼び出しを転送する場合があります。
  この :variable:`CMAKE_FIND_PACKAGE_REDIRECTS_DIR` で Config ファイルが見つからなかったら、以下で説明する手順に従って検索処理を続行します。

CMake は、任意のパッケージで利用できる Prefix を含むインストール先の集合を定義しています。
それぞれの Prefix を含むインストール先のディレクトリで Config ファイルが検索されます。
以下の表に、この検索対象のディレクトリを示します。
各ディレクトリが、ホストのどのプラットフォームに準拠したインストール・ツリーであるかを、Windows (``W``)、UNIX (``U``)、または Apple (``A``) で示します：

==================================================================== ==========
 インストール先                                                       サポート
==================================================================== ==========
 ``<prefix>/``                                                          W
 ``<prefix>/(cmake|CMake)/``                                            W
 ``<prefix>/<name>*/``                                                  W
 ``<prefix>/<name>*/(cmake|CMake)/``                                    W
 ``<prefix>/<name>*/(cmake|CMake)/<name>*/`` [#]_                       W
 ``<prefix>/(lib/<arch>|lib*|share)/cmake/<name>*/``                    U
 ``<prefix>/(lib/<arch>|lib*|share)/<name>*/``                          U
 ``<prefix>/(lib/<arch>|lib*|share)/<name>*/(cmake|CMake)/``            U
 ``<prefix>/<name>*/(lib/<arch>|lib*|share)/cmake/<name>*/``            W/U
 ``<prefix>/<name>*/(lib/<arch>|lib*|share)/<name>*/``                  W/U
 ``<prefix>/<name>*/(lib/<arch>|lib*|share)/<name>*/(cmake|CMake)/``    W/U
==================================================================== ==========

.. [#] .. versionadded:: 3.25

Apple の :prop_tgt:`FRAMEWORK` と :prop_tgt:`BUNDLE` をサポートしているシステムでは、次のインストール先で Config ファイルを含むフレームワークやアプリのバンドル [#hint_for_framework_and_bundle_of_ios]_ を探します：

=========================================================== ==========
 インストール先                                              サポート
=========================================================== ==========
 ``<prefix>/<name>.framework/Resources/``                      A
 ``<prefix>/<name>.framework/Resources/CMake/``                A
 ``<prefix>/<name>.framework/Versions/*/Resources/``           A
 ``<prefix>/<name>.framework/Versions/*/Resources/CMake/``     A
 ``<prefix>/<name>.app/Contents/Resources/``                   A
 ``<prefix>/<name>.app/Contents/Resources/CMake/``             A
=========================================================== ==========

.. [#hint_for_framework_and_bundle_of_ios] 「`Frameworkとは＠Qiita <https://qiita.com/gdate/items/b49ef26824504bb61856#framework%E3%81%A8%E3%81%AF>`_」参照。

全てのインストール先で ``<name>`` は大小文字を区別せず扱われ、指定した名前（``<PackageName>`` または ``NAMES`` オプションで指定した名前）で置き換えられます。

CMake 変数の :variable:`CMAKE_LIBRARY_ARCHITECTURE`  に ``<arch>`` がセットされている場合は ``lib/<arch>`` を含むディレクトリになります。
``lib*`` は、 ``lib64``、``lib32``、``libx32`` または ``lib`` のいずれかが1つ以上含まれます（検索はこの順番で行われます）。

* :prop_gbl:`FIND_LIBRARY_USE_LIB64_PATHS` というグローバルなプロパティを ``TRUE`` にすると、64bit のプラットフォームで ``lib64`` を持つディレクトリを検索する
* :prop_gbl:`FIND_LIBRARY_USE_LIB32_PATHS` というグローバルなプロパティを ``TRUE`` にすると、32bit のプラットフォームで ``lib32`` を持つディレクトリを検索する
* :prop_gbl:`FIND_LIBRARY_USE_LIBX32_PATHS` というグローバルなプロパティを ``TRUE`` にすると、x32 ABI を使うプラットフォームで ``libx32`` を持つディレクトリを検索する。
* ``lib`` を持つディレクトリは常に検索する。

.. versionchanged:: 3.24
  ホストが Windows 系プラットフォームの場合、:ref:`dedicated syntax <Find Using Windows Registry>` を使用して ``HINTS`` と ``PATHS`` オプションで指定した部分的なディレクトリをレジストリの照会に含めることができます。
  このような仕様は Windows 系以外のプラットフォームで無視されます。

``REGISTRY_VIEW``
  .. versionadded:: 3.24
     ``PATHS`` と ``HINTS`` オプションで指定した Windows のレジストリの照会を管理できるようになった。

  .. include:: FIND_XXX_REGISTRY_VIEW.txt

``PATH_SUFFIXES`` オプションを指定すると、その文字列が (``W``) または (``U``) のインストール先のディレクトリに追加されます。

CMake は、各インストール先のディレクトリが Config ファイルを提供するパッケージと連携して使われることを期待します。
表中で ``W`` が付いているディレクトリは Windows 系プラットフォームへのインストールを対象としたもので、その ``<prefix>`` はアプリケーションがインストールされた先頭ディレクトリを指す場合があります。
表中で ``U`` が付いているディレクトリは UNIX 系プラットフォームへのインストールを対象としたもので、``<prefix>`` は複数のパッケージで共有されるのが普通です。
表中で ``W`` または ``U`` が付いているディレクトリは全てのプラットフォームで検索されます。
表中で ``A`` が付いているディレクトリは Apple 系プラットフォームへのインストールを対象としています。
このプラットフォームでは、CMake 変数の :variable:`CMAKE_FIND_FRAMEWORK` と :variable:`CMAKE_FIND_APPBUNDLE` によって検索する優先順位が決まります。

Config ファイルを検索する際、インストール先の ``<prefix>`` は次の手順で決定します。
なお ``NO_DEFAULT_PATH`` オプションを指定すると、``NO_*`` 系のオプションが全て有効になります。

1. ``<PackageName>`` 専用のパスを ``<prefix>`` にする。
   :policy:`CMP0074` のポリシーを参照のこと。

   .. versionadded:: 3.12

   具体的には、次の変数で指定されたパスを ``<prefix>`` にして順番に検索していく：

   a. CMake 変数の :variable:`<PackageName>_ROOT` （``<PackageName>`` は大文字・小文字を区別したパッケージ名）。

   b. CMake 変数の :variable:`<PACKAGENAME>_ROOT` （``<PACKAGENAME>`` は大文字のパッケージ名）。
      :policy:`CMP0144` のポリシーを参照のこと。

      .. versionadded:: 3.27

   c. 環境変数の :envvar:`<PackageName>_ROOT` （``<PackageName>`` は大文字・小文字を区別したパッケージ名）。

   d. 環境変数の :envvar:`<PACKAGENAME>_ROOT` （``<PACKAGENAME>`` は大文字のパッケージ名）。
      :policy:`CMP0144` のポリシーを参照のこと。

      .. versionadded:: 3.27

   パッケージの root 変数はスタックとして保持するので、「:ref:`Find モジュール <Libraries not Providing Config-file Packages>`」の中で、このコマンドが呼び出された場合は、まずこのパッケージ固有のディレクトリを検索し、そのあとに Find モジュールからも検索される。
   このステップは、``NO_PACKAGE_ROOT_PATH`` オプションを指定するか、または CMake 変数の :variable:`CMAKE_FIND_USE_PACKAGE_ROOT_PATH` を ``FALSE`` にセットした時はスキップする。

2. キャッシュ変数で指定したパスを ``<prefix>`` にする。
   これは :option:`-DVAR=VALUE <cmake -D>` オプションを指定した :manual:`cmake(1)` コマンドラインでの使用を意図している。
   ``VALUE`` はセミコロンで区切った :ref:`リスト <CMake Language Lists>` として解釈する。
   このステップは、``NO_CMAKE_PATH`` オプションを指定するか、または CMake 変数の :variable:`CMAKE_FIND_USE_CMAKE_PATH` を ``FALSE`` にするとスキップできる。

   * :variable:`CMAKE_PREFIX_PATH`
   * :variable:`CMAKE_FRAMEWORK_PATH`
   * :variable:`CMAKE_APPBUNDLE_PATH`

3. CMake 専用の環境変数で指定したパスを ``<prefix>`` にする。
   これはユーザが導入したシェルスクリプトの中でインストール先のディレクトリを指定する場合を想定しており、ホストのプラットフォームで有効なパスの区切り文字（Windows 系プラットフォームの場合は ``;``、UNIX 系プラットフォームの場合は ``:``）と使うこと。
   このステップは、``NO_CMAKE_ENVIRONMENT_PATH`` オプションを指定するか、または CMake 変数の :variable:`CMAKE_FIND_USE_CMAKE_ENVIRONMENT_PATH` を ``FALSE`` にするとスキップできる。

   * ``<PackageName>_DIR``
   * :envvar:`CMAKE_PREFIX_PATH`
   * :envvar:`CMAKE_FRAMEWORK_PATH`
   * :envvar:`CMAKE_APPBUNDLE_PATH`

4. Search paths specified by the ``HINTS`` option.
   These should be paths computed by system introspection, such as a hint provided by the location of another item already found.
   Hard-coded guesses should be specified with the ``PATHS`` option.

5. Search the standard system environment variables.
   This can be skipped if ``NO_SYSTEM_ENVIRONMENT_PATH`` is passed  or by setting the :variable:`CMAKE_FIND_USE_SYSTEM_ENVIRONMENT_PATH` to ``FALSE``.
   Path entries ending in ``/bin`` or ``/sbin`` are automatically converted to their parent directories:

   * ``PATH``

6. Search paths stored in the CMake :ref:`User Package Registry`.
   This can be skipped if ``NO_CMAKE_PACKAGE_REGISTRY`` is passed or by setting the variable :variable:`CMAKE_FIND_USE_PACKAGE_REGISTRY` to ``FALSE`` or the deprecated variable :variable:`CMAKE_FIND_PACKAGE_NO_PACKAGE_REGISTRY` to ``TRUE``.

   See the :manual:`cmake-packages(7)` manual for details on the user package registry.

7. Search cmake variables defined in the Platform files for the current system.
   The searching of :variable:`CMAKE_INSTALL_PREFIX` and :variable:`CMAKE_STAGING_PREFIX` can be skipped if ``NO_CMAKE_INSTALL_PREFIX`` is passed or by setting the :variable:`CMAKE_FIND_USE_INSTALL_PREFIX` to ``FALSE``.
   All these locations can be skipped if ``NO_CMAKE_SYSTEM_PATH`` is passed or by setting the :variable:`CMAKE_FIND_USE_CMAKE_SYSTEM_PATH` to ``FALSE``:

   * :variable:`CMAKE_SYSTEM_PREFIX_PATH`
   * :variable:`CMAKE_SYSTEM_FRAMEWORK_PATH`
   * :variable:`CMAKE_SYSTEM_APPBUNDLE_PATH`

   The platform paths that these variables contain are locations that typically include installed software. An example being ``/usr/local`` for UNIX based platforms.

8. Search paths stored in the CMake :ref:`System Package Registry`.
   This can be skipped if ``NO_CMAKE_SYSTEM_PACKAGE_REGISTRY`` is passed or by setting the :variable:`CMAKE_FIND_USE_SYSTEM_PACKAGE_REGISTRY` variable to ``FALSE`` or the deprecated variable :variable:`CMAKE_FIND_PACKAGE_NO_SYSTEM_PACKAGE_REGISTRY` to ``TRUE``.

   See the :manual:`cmake-packages(7)` manual for details on the system package registry.

9. Search paths specified by the ``PATHS`` option.
   These are typically hard-coded guesses.

The :variable:`CMAKE_IGNORE_PATH`, :variable:`CMAKE_IGNORE_PREFIX_PATH`, :variable:`CMAKE_SYSTEM_IGNORE_PATH` and :variable:`CMAKE_SYSTEM_IGNORE_PREFIX_PATH` variables can also cause some of the above locations to be ignored.

.. versionadded:: 3.16
   Added the ``CMAKE_FIND_USE_<CATEGORY>`` variables to globally disable various search locations.

.. include:: FIND_XXX_ROOT.txt
.. include:: FIND_XXX_ORDER.txt

By default the value stored in the result variable will be the path at which the file is found.
The :variable:`CMAKE_FIND_PACKAGE_RESOLVE_SYMLINKS` variable may be set to ``TRUE`` before calling ``find_package`` in order to resolve symbolic links and store the real path to the file.

Every non-REQUIRED ``find_package`` call can be disabled or made REQUIRED:

* Setting the :variable:`CMAKE_DISABLE_FIND_PACKAGE_<PackageName>` variable  to ``TRUE`` disables the package.
  This also disables redirection to a package provided by :module:`FetchContent`.

* Setting the :variable:`CMAKE_REQUIRE_FIND_PACKAGE_<PackageName>` variable to ``TRUE`` makes the package REQUIRED.

Setting both variables to ``TRUE`` simultaneously is an error.

.. _`version selection`:

Config Mode Version Selection
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::
  When Config mode is used, this version selection process is applied regardless of whether the :ref:`full <full signature>` or :ref:`basic <basic signature>` signature was given.

When the ``[version]`` argument is given, Config mode will only find a version of the package that claims compatibility with the requested version (see :ref:`format specification <FIND_PACKAGE_VERSION_FORMAT>`).
If the ``EXACT`` option is given, only a version of the package claiming an exact match of the requested version may be found.
CMake does not establish any convention for the meaning of version numbers.
Package version numbers are checked by "version" files provided by the packages themselves or by :module:`FetchContent`.
For a candidate package configuration file ``<config-file>.cmake`` the corresponding version file is located next to it and named either ``<config-file>-version.cmake`` or
``<config-file>Version.cmake``.
If no such version file is available then the configuration file is assumed to not be compatible with any requested version.
A basic version file containing generic version matching code can be created using the :module:`CMakePackageConfigHelpers` module.
When a version file is found it is loaded to check the requested version number.
The version file is loaded in a nested scope in which the following variables have been defined:

``PACKAGE_FIND_NAME``
  The ``<PackageName>``
``PACKAGE_FIND_VERSION``
  Full requested version string
``PACKAGE_FIND_VERSION_MAJOR``
  Major version if requested, else 0
``PACKAGE_FIND_VERSION_MINOR``
  Minor version if requested, else 0
``PACKAGE_FIND_VERSION_PATCH``
  Patch version if requested, else 0
``PACKAGE_FIND_VERSION_TWEAK``
  Tweak version if requested, else 0
``PACKAGE_FIND_VERSION_COUNT``
  Number of version components, 0 to 4

When a version range is specified, the above version variables will hold values based on the lower end of the version range.
This is to preserve compatibility with packages that have not been implemented to expect version ranges.
In addition, the version range will be described by the following variables:

``PACKAGE_FIND_VERSION_RANGE``
  Full requested version range string
``PACKAGE_FIND_VERSION_RANGE_MIN``
  This specifies whether the lower end point of the version range should be included or excluded.
  Currently, the only supported value for this variable is ``INCLUDE``.
``PACKAGE_FIND_VERSION_RANGE_MAX``
  This specifies whether the upper end point of the version range should be included or excluded.
  The supported values for this variable are  ``INCLUDE`` and ``EXCLUDE``.

``PACKAGE_FIND_VERSION_MIN``
  Full requested version string of the lower end point of the range
``PACKAGE_FIND_VERSION_MIN_MAJOR``
  Major version of the lower end point if requested, else 0
``PACKAGE_FIND_VERSION_MIN_MINOR``
  Minor version of the lower end point if requested, else 0
``PACKAGE_FIND_VERSION_MIN_PATCH``
  Patch version of the lower end point if requested, else 0
``PACKAGE_FIND_VERSION_MIN_TWEAK``
  Tweak version of the lower end point if requested, else 0
``PACKAGE_FIND_VERSION_MIN_COUNT``
  Number of version components of the lower end point, 0 to 4

``PACKAGE_FIND_VERSION_MAX``
  Full requested version string of the upper end point of the range 
``PACKAGE_FIND_VERSION_MAX_MAJOR``
  Major version of the upper end point if requested, else 0
``PACKAGE_FIND_VERSION_MAX_MINOR``
  Minor version of the upper end point if requested, else 0
``PACKAGE_FIND_VERSION_MAX_PATCH``
  Patch version of the upper end point if requested, else 0
``PACKAGE_FIND_VERSION_MAX_TWEAK``
  Tweak version of the upper end point if requested, else 0
``PACKAGE_FIND_VERSION_MAX_COUNT``
  Number of version components of the upper end point, 0 to 4

Regardless of whether a single version or a version range is specified, the variable ``PACKAGE_FIND_VERSION_COMPLETE`` will be defined and will hold the full requested version string as specified.

The version file checks whether it satisfies the requested version and sets these variables:

``PACKAGE_VERSION``
  Full provided version string
``PACKAGE_VERSION_EXACT``
  True if version is exact match
``PACKAGE_VERSION_COMPATIBLE``
  True if version is compatible
``PACKAGE_VERSION_UNSUITABLE``
  True if unsuitable as any version

These variables are checked by the ``find_package`` command to determine whether the configuration file provides an acceptable version.
They are not available after the ``find_package`` call returns.  If the version is acceptable the following variables are set:

``<PackageName>_VERSION``
  Full provided version string
``<PackageName>_VERSION_MAJOR``
  Major version if provided, else 0
``<PackageName>_VERSION_MINOR``
  Minor version if provided, else 0
``<PackageName>_VERSION_PATCH``
  Patch version if provided, else 0
``<PackageName>_VERSION_TWEAK``
  Tweak version if provided, else 0
``<PackageName>_VERSION_COUNT``
  Number of version components, 0 to 4

and the corresponding package configuration file is loaded.
When multiple package configuration files are available whose version files claim compatibility with the version requested it is unspecified which one is chosen: unless the variable :variable:`CMAKE_FIND_PACKAGE_SORT_ORDER` is set no attempt is made to choose a highest or closest version number.

To control the order in which ``find_package`` checks for compatibility use the two variables :variable:`CMAKE_FIND_PACKAGE_SORT_ORDER` and :variable:`CMAKE_FIND_PACKAGE_SORT_DIRECTION`.
For instance in order to select the highest version one can set

.. code-block:: cmake

  SET(CMAKE_FIND_PACKAGE_SORT_ORDER NATURAL)
  SET(CMAKE_FIND_PACKAGE_SORT_DIRECTION DEC)

before calling ``find_package``.

Package File Interface Variables
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When loading a find module or package configuration file ``find_package`` defines variables to provide information about the call arguments (and restores their original state before returning):

``CMAKE_FIND_PACKAGE_NAME``
  The ``<PackageName>`` which is searched for
``<PackageName>_FIND_REQUIRED``
  True if ``REQUIRED`` option was given
``<PackageName>_FIND_QUIETLY``
  True if ``QUIET`` option was given
``<PackageName>_FIND_REGISTRY_VIEW``
  The requested view if ``REGISTRY_VIEW`` option was given
``<PackageName>_FIND_VERSION``
  Full requested version string
``<PackageName>_FIND_VERSION_MAJOR``
  Major version if requested, else 0
``<PackageName>_FIND_VERSION_MINOR``
  Minor version if requested, else 0
``<PackageName>_FIND_VERSION_PATCH``
  Patch version if requested, else 0
``<PackageName>_FIND_VERSION_TWEAK``
  Tweak version if requested, else 0
``<PackageName>_FIND_VERSION_COUNT``
  Number of version components, 0 to 4
``<PackageName>_FIND_VERSION_EXACT``
  True if ``EXACT`` option was given
``<PackageName>_FIND_COMPONENTS``
  List of specified components (required and optional)
``<PackageName>_FIND_REQUIRED_<c>``
  True if component ``<c>`` is required,
  false if component ``<c>`` is optional

When a version range is specified, the above version variables will hold values based on the lower end of the version range.
This is to preserve compatibility with packages that have not been implemented to expect version ranges.
In addition, the version range will be described by the following variables:

``<PackageName>_FIND_VERSION_RANGE``
  Full requested version range string
``<PackageName>_FIND_VERSION_RANGE_MIN``
  This specifies whether the lower end point of the version range is included or excluded.  Currently, ``INCLUDE`` is the only supported value.
``<PackageName>_FIND_VERSION_RANGE_MAX``
  This specifies whether the upper end point of the version range is included or excluded.
  The possible values for this variable are ``INCLUDE`` or ``EXCLUDE``.

``<PackageName>_FIND_VERSION_MIN``
  Full requested version string of the lower end point of the range
``<PackageName>_FIND_VERSION_MIN_MAJOR``
  Major version of the lower end point if requested, else 0
``<PackageName>_FIND_VERSION_MIN_MINOR``
  Minor version of the lower end point if requested, else 0
``<PackageName>_FIND_VERSION_MIN_PATCH``
  Patch version of the lower end point if requested, else 0
``<PackageName>_FIND_VERSION_MIN_TWEAK``
  Tweak version of the lower end point if requested, else 0
``<PackageName>_FIND_VERSION_MIN_COUNT``
  Number of version components of the lower end point, 0 to 4

``<PackageName>_FIND_VERSION_MAX``
  Full requested version string of the upper end point of the range
``<PackageName>_FIND_VERSION_MAX_MAJOR``
  Major version of the upper end point if requested, else 0
``<PackageName>_FIND_VERSION_MAX_MINOR``
  Minor version of the upper end point if requested, else 0
``<PackageName>_FIND_VERSION_MAX_PATCH``
  Patch version of the upper end point if requested, else 0
``<PackageName>_FIND_VERSION_MAX_TWEAK``
  Tweak version of the upper end point if requested, else 0
``<PackageName>_FIND_VERSION_MAX_COUNT``
  Number of version components of the upper end point, 0 to 4

Regardless of whether a single version or a version range is specified, the variable ``<PackageName>_FIND_VERSION_COMPLETE`` will be defined and will hold the full requested version string as specified.

In Module mode the loaded find module is responsible to honor the request detailed by these variables; see the find module for details.
In Config mode ``find_package`` handles ``REQUIRED``, ``QUIET``, and ``[version]`` options automatically but leaves it to the package configuration file to handle components in a way that makes sense
for the package.
The package configuration file may set ``<PackageName>_FOUND`` to false to tell ``find_package`` that component requirements are not satisfied.
