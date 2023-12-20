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
  ``REGISTRY_VIEW`` オプションには、どのレジストリ・ビューをクエリするかを指定します。
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
    ここに ``<`` を指定すると ``versionMax`` は含めない範囲になる。
    このバージョン範囲は CMake のバージョン 3.19 以降でのサポートになる。

``EXACT`` オプションは、完全に一致するバージョンを要求します。
このオプションは「バージョンを任意の範囲で指定する」場合と一緒に指定できません（指定しても無視されます）。

このコマンドを「:ref:`Find Modules`」内で再起呼び出しした時に ``[version]`` や ``COMPONENTS`` が指定されていない場合、これらに対応するオプションは、このコマンドをプロジェクト（外部）から呼びだした時のオプションが自動的に転送されます（``[version]`` の ``EXACT`` も同様 ）。
このバージョン制約の機能は、現在はパッケージ単位でのみのサポートです（この下の `Version Selection`_ セクションを参照のこと ）。
つまりバージョンの範囲が指定されても、パッケージ側で単一のバージョンを期待する設計になっている場合、バージョン範囲の上限値を無視して、バージョン範囲の下限値だけを受け入れるようになっています。

``NO_POLICY_SCOPE`` オプションの詳細については、:command:`cmake_policy`  コマンドのドキュメントを参照して下さい。

.. versionadded:: 3.24
  ``BYPASS_PROVIDER`` オプションは、「:ref:`依存関係のプロバイダ <dependency_providers>`」が ``find_package()`` コマンドを呼び出した時にだけ指定できます。
  このプロバイダは CMake 内の ``find_package()`` の実装を直接呼び出し、``find_package()`` の実装で呼び出されることがないようにします。
  CMake の将来のバージョンでは、このオプションをプロバイダ以外から使おうとすると、致命的なエラーで停止する可能性があります。

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

``CONFIG`` オプション、またはこれと同じ意味を持つ ``NO_MODULE`` オプション、あるいは 「:ref:`basic signature`」で指定されていないオプションを指定すると、すべて強制的に「純粋」 Config モードによる検索を行います。
この「純粋」 Config モードは、Module モードを使った検索はスキップします（スキップしたあとは Config モードで検索を継続します）。

Config モードによる検索は、パッケージが提供している「:ref:`Config ファイル <Config File Packages>`」を見つけようとします。
CMake は、見つかった Config ファイルとその場所（ディレクトリ）を格納するキャッシュ変数の ``<PackageName>_DIR``  を自動的に生成します。
このコマンドは、デフォルトで ``<PackageName>`` という名前のパッケージを探します。
なお ``NAMES`` オプションを指定すると、このオプションに渡した名前（文字列）を ``<PackageName>`` の代わりに使います。
この名前は、コマンドの呼び出しを :module:`FetchContent` モジュールが提供したパッケージに転送するかを決める際にも使われます。

このコマンドは、指定された ``<PackageName>``（名前）ごとに ``<PackageName>Config.cmake`` または ``<lowercasePackageName>-config.cmake`` のファイルを探します。

A replacement set of possible configuration file names may be given using the ``CONFIGS`` option.
The :ref:`search procedure` is specified below.
Once found, any :ref:`version constraint <version selection>` is checked,
and if satisfied, the configuration file is read and processed by CMake.
Since the file is provided by the package it already knows the location of package contents.
The full path to the configuration file is stored in the cmake variable ``<PackageName>_CONFIG``.
このコマンドは、指定された名前ごとに、<PackageName>Config.cmake または < lowercasePackageName>-config.cmake というファイルを検索します。 CONFIGS オプションを使用して、可能な構成ファイル名の置換セットを指定できます。 Config Mode の検索手順を以下に示します。 見つかった場合は、バージョン制約がチェックされ、満たされている場合は、構成ファイルが CMake によって読み取られて処理されます。 ファイルはパッケージによって提供されるため、パッケージの内容の場所がすでにわかっています。 構成ファイルへの絶対パスは、cmake 変数 <PackageName>_CONFIG に保存されます。

All configuration files which have been considered by CMake while searching for the package with an appropriate version are stored in the ``<PackageName>_CONSIDERED_CONFIGS`` variable, and the associated versions in the ``<PackageName>_CONSIDERED_VERSIONS`` variable.

If the package configuration file cannot be found CMake will generate an error describing the problem unless the ``QUIET`` argument is specified.
If ``REQUIRED`` is specified and the package is not found a fatal error is generated and the configure step stops executing.
If ``<PackageName>_DIR`` has been set to a directory not containing a configuration file CMake will ignore it and search from scratch.

Package maintainers providing CMake package configuration files are encouraged to name and install them such that the :ref:`search procedure` outlined below will find them without requiring use of additional options.

.. _`search procedure`:

Config Mode Search Procedure
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. note::
  When Config mode is used, this search procedure is applied regardless of  whether the :ref:`full <full signature>` or :ref:`basic <basic signature>` signature was given.

.. versionadded:: 3.24
  All calls to ``find_package()`` (even in Module mode) first look for a config package file in the :variable:`CMAKE_FIND_PACKAGE_REDIRECTS_DIR` directory.
  The :module:`FetchContent` module, or even the project itself, may write files to that location to redirect ``find_package()`` calls to content already provided by the project.
  If no config package file is found in that location, the search proceeds with the logic described below.

CMake constructs a set of possible installation prefixes for the package.
Under each prefix several directories are searched for a configuration file.
The tables below show the directories searched.
Each entry is meant for installation trees following Windows (``W``), UNIX (``U``), or Apple (``A``) conventions:

==================================================================== ==========
 Entry                                                               Convention
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

On systems supporting macOS :prop_tgt:`FRAMEWORK` and :prop_tgt:`BUNDLE`, the following directories are searched for Frameworks or Application Bundles containing a configuration file:

=========================================================== ==========
 Entry                                                      Convention
=========================================================== ==========
 ``<prefix>/<name>.framework/Resources/``                      A
 ``<prefix>/<name>.framework/Resources/CMake/``                A
 ``<prefix>/<name>.framework/Versions/*/Resources/``           A
 ``<prefix>/<name>.framework/Versions/*/Resources/CMake/``     A
 ``<prefix>/<name>.app/Contents/Resources/``                   A
 ``<prefix>/<name>.app/Contents/Resources/CMake/``             A
=========================================================== ==========

In all cases the ``<name>`` is treated as case-insensitive and corresponds to any of the names specified (``<PackageName>`` or names given by ``NAMES``).

Paths with ``lib/<arch>`` are enabled if the :variable:`CMAKE_LIBRARY_ARCHITECTURE` variable is set.
``lib*`` includes one or more of the values ``lib64``, ``lib32``, ``libx32`` or ``lib`` (searched in that order).

* Paths with ``lib64`` are searched on 64 bit platforms if the :prop_gbl:`FIND_LIBRARY_USE_LIB64_PATHS` property is set to ``TRUE``.
* Paths with ``lib32`` are searched on 32 bit platforms if the :prop_gbl:`FIND_LIBRARY_USE_LIB32_PATHS` property is set to ``TRUE``.
* Paths with ``libx32`` are searched on platforms using the x32 ABI if the :prop_gbl:`FIND_LIBRARY_USE_LIBX32_PATHS` property is set to ``TRUE``.
* The ``lib`` path is always searched.

.. versionchanged:: 3.24
  On ``Windows`` platform, it is possible to include registry queries as part of the directories specified through ``HINTS`` and ``PATHS`` keywords, using a :ref:`dedicated syntax <Find Using Windows Registry>`.
  Such specifications will be ignored on all other platforms.

.. versionadded:: 3.24
  ``REGISTRY_VIEW`` can be specified to manage ``Windows`` registry queries specified as part of ``PATHS`` and ``HINTS``.

.. include:: FIND_XXX_REGISTRY_VIEW.txt

If ``PATH_SUFFIXES`` is specified, the suffixes are appended to each (``W``) or (``U``) directory entry one-by-one.

This set of directories is intended to work in cooperation with projects that provide configuration files in their installation trees.
Directories above marked with (``W``) are intended for installations on Windows where the prefix may point at the top of an application's installation directory.
Those marked with (``U``) are intended for installations on UNIX platforms where the prefix is shared by multiple packages.
This is merely a convention, so all (``W``) and (``U``) directories are still searched on all platforms.
Directories marked with (``A``) are intended for installations on Apple platforms.
The :variable:`CMAKE_FIND_FRAMEWORK` and :variable:`CMAKE_FIND_APPBUNDLE` variables determine the order of preference.

The set of installation prefixes is constructed using the following steps.
If ``NO_DEFAULT_PATH`` is specified all ``NO_*`` options are enabled.

1. Search prefixes unique to the current ``<PackageName>`` being found.
   See policy :policy:`CMP0074`.

   .. versionadded:: 3.12

   Specifically, search prefixes specified by the following variables, in order:

   a. :variable:`<PackageName>_ROOT` CMake variable,  where ``<PackageName>`` is the case-preserved package name.

   b. :variable:`<PACKAGENAME>_ROOT` CMake variable,  where ``<PACKAGENAME>`` is the upper-cased package name.
      See policy :policy:`CMP0144`.

      .. versionadded:: 3.27

   c. :envvar:`<PackageName>_ROOT` environment variable, where ``<PackageName>`` is the case-preserved package name.

   d. :envvar:`<PACKAGENAME>_ROOT` environment variable, where ``<PACKAGENAME>`` is the upper-cased package name.
      See policy :policy:`CMP0144`.

      .. versionadded:: 3.27

   The package root variables are maintained as a stack so if called from within a find module, root paths from the parent's find module will also be searched after paths for the current package.
   This can be skipped if ``NO_PACKAGE_ROOT_PATH`` is passed or by setting the :variable:`CMAKE_FIND_USE_PACKAGE_ROOT_PATH` to ``FALSE``.

2. Search paths specified in cmake-specific cache variables.
   These are intended to be used on the command line with a :option:`-DVAR=VALUE <cmake -D>`.
   The values are interpreted as :ref:`semicolon-separated lists <CMake Language Lists>`.
   This can be skipped if ``NO_CMAKE_PATH`` is passed or by setting the :variable:`CMAKE_FIND_USE_CMAKE_PATH` to ``FALSE``:

   * :variable:`CMAKE_PREFIX_PATH`
   * :variable:`CMAKE_FRAMEWORK_PATH`
   * :variable:`CMAKE_APPBUNDLE_PATH`

3. Search paths specified in cmake-specific environment variables.
   These are intended to be set in the user's shell configuration, and therefore use the host's native path separator (``;`` on Windows and ``:`` on UNIX).
   This can be skipped if ``NO_CMAKE_ENVIRONMENT_PATH`` is passed or by setting the :variable:`CMAKE_FIND_USE_CMAKE_ENVIRONMENT_PATH` to ``FALSE``:

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
