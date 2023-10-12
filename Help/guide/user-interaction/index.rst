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

KDevelop4、QtCreator や CLion  のような一部の IDE は CMake ベースのビルドシステムをネイティブでサポートしています。

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

The "Configure" button triggers a new dialog to select the CMake generator to use.

.. image:: GUI-Configure-Dialog.png
   :alt: Configuring a generator

All generators available on the command line are also available in :manual:`cmake-gui(1)`.

.. image:: GUI-Choose-Generator.png
   :alt: Choosing a generator

When choosing a Visual Studio generator, further options are available to set an architecture to generate for.

.. image:: VS-Choose-Arch.png
   :alt: Choosing an architecture for Visual Studio generators

.. _`Setting Build Variables`:

ビルド用の変数をセットする
==========================

Software projects often require variables to be set on the command line when invoking CMake.
Some of the most commonly used CMake variables are listed in the table below:

========================================== ============================================================
 Variable                                   Meaning
========================================== ============================================================
 :variable:`CMAKE_PREFIX_PATH`              Path to search for
                                            :guide:`dependent packages <Using Dependencies Guide>`
 :variable:`CMAKE_MODULE_PATH`              Path to search for additional CMake modules
 :variable:`CMAKE_BUILD_TYPE`               Build configuration, such as
                                            ``Debug`` or ``Release``, determining
                                            debug/optimization flags.  This is only
                                            relevant for single-configuration buildsystems such
                                            as ``Makefile`` and ``Ninja``.  Multi-configuration
                                            buildsystems such as those for Visual Studio and Xcode
                                            ignore this setting.
 :variable:`CMAKE_INSTALL_PREFIX`           Location to install the
                                            software to with the
                                            ``install`` build target
 :variable:`CMAKE_TOOLCHAIN_FILE`           File containing cross-compiling
                                            data such as
                                            :manual:`toolchains and sysroots <cmake-toolchains(7)>`.
 :variable:`BUILD_SHARED_LIBS`              Whether to build shared
                                            instead of static libraries
                                            for :command:`add_library`
                                            commands used without a type
 :variable:`CMAKE_EXPORT_COMPILE_COMMANDS`  Generate a ``compile_commands.json``
                                            file for use with clang-based tools
========================================== ============================================================

Other project-specific variables may be available to control builds, such as enabling or disabling components of the project.

There is no convention provided by CMake for how such variables are named between different provided buildsystems, except that variables with the prefix ``CMAKE_`` usually refer to options provided by CMake itself and should not be used in third-party options, which should use their own prefix instead.
The :manual:`cmake-gui(1)` tool can display options in groups defined by their prefix, so it makes sense for third parties to ensure that they use a self-consistent prefix.


コマンドラインから変数をセットする
----------------------------------

CMake variables can be set on the command line either when creating the initial build:

.. code-block:: console

    $ mkdir build
    $ cd build
    $ cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Debug

or later on a subsequent invocation of :manual:`cmake(1)`:

.. code-block:: console

    $ cd build
    $ cmake . -DCMAKE_BUILD_TYPE=Debug

The :option:`-U <cmake -U>` flag may be used to unset variables on the :manual:`cmake(1)` command line:

.. code-block:: console

    $ cd build
    $ cmake . -UMyPackage_DIR

A CMake buildsystem which was initially created on the command line can be modified using the :manual:`cmake-gui(1)` and vice-versa.

The :manual:`cmake(1)` tool allows specifying a file to use to populate the initial cache using the :option:`-C <cmake -C>` option.
This can be useful to simplify commands and scripts which repeatedly require the same cache entries.


cmake-gui で変数をセットする
----------------------------

Variables may be set in the cmake-gui using the "Add Entry" button.
This triggers a new dialog to set the value of the variable.

.. image:: GUI-Add-Entry.png
   :alt: Editing a cache entry

The main view of the :manual:`cmake-gui(1)` user interface can be used to edit existing variables.

CMake キャッシュ
----------------

When CMake is executed, it needs to find the locations of compilers, tools and dependencies.
It also needs to be able to consistently re-generate a buildsystem to use the same compile/link flags and paths to dependencies.
Such parameters are also required to be configurable by the user because they are paths and options specific to the users system.

When it is first executed, CMake generates a ``CMakeCache.txt`` file in the build directory containing key-value pairs for such artifacts.
The cache file can be viewed or edited by the user by running the :manual:`cmake-gui(1)` or :manual:`ccmake(1)` tool.
The tools provide an interactive interface for re-configuring the provided software and re-generating the buildsystem, as is needed after editing cached values.
Each cache entry may have an associated short help text which is displayed in the user interface tools.

The cache entries may also have a type to signify how it should be presented in the user interface.
For example, a cache entry of type ``BOOL`` can be edited by a checkbox in a user interface, a ``STRING`` can be edited in a text field, and a ``FILEPATH`` while similar to a ``STRING`` should also provide a way to locate filesystem paths using a file dialog.
An entry of type ``STRING`` may provide a restricted list of allowed values which are then provided in a drop-down menu in the :manual:`cmake-gui(1)` user interface (see the :prop_cache:`STRINGS` cache property).

The CMake files shipped with a software package may also define boolean toggle options using the :command:`option` command.
The command creates a cache entry which has a help text and a default value.
Such cache entries are typically specific to the provided software and affect the configuration of the build, such as whether tests and examples are built, whether to build with exceptions enabled etc.

プリセットを使う
================

CMake understands a file, ``CMakePresets.json``, and its user-specific counterpart, ``CMakeUserPresets.json``, for saving presets for commonly-used configure settings.
These presets can set the build directory, generator, cache variables, environment variables, and other command-line options.
All of these options can be overridden by the user.
The full details of the ``CMakePresets.json`` format are listed in the :manual:`cmake-presets(7)` manual.

コマンドラインからプリセットを使う
----------------------------------

When using the :manual:`cmake(1)` command line tool, a preset can be invoked by using the :option:`--preset <cmake --preset>` option.
If :option:`--preset <cmake --preset>` is specified, the generator and build directory are not required, but can be specified to override them.
For example, if you have the following ``CMakePresets.json`` file:

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

and you run the following:

.. code-block:: console

  cmake -S /path/to/source --preset=ninja-release

This will generate a build directory in ``/path/to/source/build/ninja-release`` with the :generator:`Ninja` generator, and with :variable:`CMAKE_BUILD_TYPE` set to ``Release``.

If you want to see the list of available presets, you can run:

.. code-block:: console

  cmake -S /path/to/source --list-presets

This will list the presets available in ``/path/to/source/CMakePresets.json`` and ``/path/to/source/CMakeUsersPresets.json`` without generating a build tree.

cmake-gui でプリセットを使う
----------------------------

If a project has presets available, either through ``CMakePresets.json`` or ``CMakeUserPresets.json``, the list of presets will appear in a drop-down menu in :manual:`cmake-gui(1)` between the source directory and the binary directory.
Choosing a preset sets the binary directory, generator, environment variables, and cache variables, but all of these options can be overridden after a preset is selected.

ビルドシステムを呼び出す
========================

After generating the buildsystem, the software can be built by invoking the particular build tool.
In the case of the IDE generators, this can involve loading the generated project file into the IDE to invoke the build.

CMake is aware of the specific build tool needed to invoke a build so in general, to build a buildsystem or project from the command line after generating, the following command may be invoked in the build directory:

.. code-block:: console

  $ cmake --build .

The :option:`--build <cmake --build>` flag enables a particular mode of operation for the :manual:`cmake(1)` tool.
It invokes the  :variable:`CMAKE_MAKE_PROGRAM` command associated with the :manual:`generator <cmake-generators(7)>`, or the build tool configured by the user.

The :option:`--build <cmake --build>` mode also accepts the parameter :option:`--target <cmake--build --target>` to specify a particular target to build, for example a particular library, executable or custom target, or a particular special target like ``install``:

.. code-block:: console

  $ cmake --build . --target myexe

The :option:`--build <cmake --build>` mode also accepts a
:option:`--config <cmake--build --config>` parameter
in the case of multi-config generators to specify which
particular configuration to build:

.. code-block:: console

  $ cmake --build . --target myexe --config Release

The :option:`--config <cmake--build --config>` option has no
effect if the generator generates a buildsystem specific
to a configuration which is chosen when invoking cmake
with the :variable:`CMAKE_BUILD_TYPE` variable.

Some buildsystems omit details of command lines invoked
during the build.  The :option:`--verbose <cmake--build --verbose>`
flag can be used to cause those command lines to be shown:

.. code-block:: console

  $ cmake --build . --target myexe --verbose

The :option:`--build <cmake --build>` mode can also pass
particular command line options to the underlying build
tool by listing them after ``--``.  This can be useful
to specify options to the build tool, such as to continue the
build after a failed job, where CMake does not
provide a high-level user interface.

For all generators, it is possible to run the underlying
build tool after invoking CMake.  For example, ``make``
may be executed after generating with the
:generator:`Unix Makefiles` generator to invoke the build,
or ``ninja`` after generating with the :generator:`Ninja`
generator etc.  The IDE buildsystems usually provide
command line tooling for building a project which can
also be invoked.


ターゲットを選択する
--------------------

Each executable and library described in the CMake files
is a build target, and the buildsystem may describe
custom targets, either for internal use, or for user
consumption, for example to create documentation.

CMake provides some built-in targets for all buildsystems
providing CMake files.

``all``
  The default target used by ``Makefile`` and ``Ninja``
  generators.  Builds all targets in the buildsystem,
  except those which are excluded by their
  :prop_tgt:`EXCLUDE_FROM_ALL` target property or
  :prop_dir:`EXCLUDE_FROM_ALL` directory property.  The
  name ``ALL_BUILD`` is used for this purpose for the
  Xcode and Visual Studio generators.
``help``
  Lists the targets available for build.  This target is
  available when using the :generator:`Unix Makefiles` or
  :generator:`Ninja` generator, and the exact output is
  tool-specific.
``clean``
  Delete built object files and other output files.  The
  ``Makefile`` based generators create a ``clean`` target
  per directory, so that an individual directory can be
  cleaned.  The ``Ninja`` tool provides its own granular
  ``-t clean`` system.
``test``
  Runs tests.  This target is only automatically available
  if the CMake files provide CTest-based tests.  See also
  `テストを実施する`_.
``install``
  Installs the software.  This target is only automatically
  available if the software defines install rules with the
  :command:`install` command.  See also
  `ソフトウェアをインストールする`_.
``package``
  Creates a binary package.  This target is only
  automatically available if the CMake files provide
  CPack-based packages.
``package_source``
  Creates a source package.  This target is only
  automatically available if the CMake files provide
  CPack-based packages.

For ``Makefile`` based systems, ``/fast`` variants of binary
build targets are provided. The ``/fast`` variants are used
to build the specified target without regard for its
dependencies.  The dependencies are not checked and
are not rebuilt if out of date.  The :generator:`Ninja`
generator is sufficiently fast at dependency checking that
such targets are not provided for that generator.

``Makefile`` based systems also provide build-targets to
preprocess, assemble and compile individual files in a
particular directory.

.. code-block:: console

  $ make foo.cpp.i
  $ make foo.cpp.s
  $ make foo.cpp.o

The file extension is built into the name of the target
because another file with the same name but a different
extension may exist.  However, build-targets without the
file extension are also provided.

.. code-block:: console

  $ make foo.i
  $ make foo.s
  $ make foo.o

In buildsystems which contain ``foo.c`` and ``foo.cpp``,
building the ``foo.i`` target will preprocess both files.

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
