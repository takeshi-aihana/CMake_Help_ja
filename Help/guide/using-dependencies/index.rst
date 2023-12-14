依存関係を利用するためのガイド
******************************

.. only:: html

   .. contents::

はじめに
========

一般的に「プロジェクト」は他のプロジェクトや、いろいろな目的、そしていろいろな成果物（*Artifacts*）に依存することがよくあります。
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
  そのような場合は、プロジェクトまたは CMake で、パッケージ向けの ``Find`` モジュールなるものを用意できます。
  通常 ``Find`` モジュールは、そのパッケージが何のファイルを配布し、それらをどのようにしてプロジェクト側に提供しているのかを解決する実装を持っています。
  通常 ``Find`` モジュールはパッケージと別々に提供されているため、検索結果の信頼性はそれほど高くありません。
  つまり、パッケージと ``Find`` モジュールは個別に保守され、異なるスケジュールに従ってリリースされる場合が多いので、簡単に情報が古くなってしまう可能性があります。

この :command:`find_package` コマンドは、受け取ったオプションに応じて、二つあるモードの一方を使うか、あるいは両方を使うかを決めます。
呼び出せるコマンドの「:ref:`basic signature`」を制限することで、両方のモードを使った依存関係の解決が可能になります。
つまり、いろいろなオプションを渡してしまうと、2つあるモードのうち1つだけに使用が制限されてしまい、このコマンドの本来の能力を発揮できないかもしれません。
この複雑なトピックについて詳細は、この :command:`find_package` のドキュメントを参照してみて下さい。

いずれのモードでも、ユーザは :manual:`cmake(1)` のコマンドライン、または :manual:`ccmake(1)` や :manual:`cmake-gui(1)` などの GUI ツールでキャッシュ変数を指定することで、パッケージを探す場所をカスタマイズすることができます。
この指定方法については「:ref:`ユーザ操作ガイド <Setting Build Variables>`」を参照して下さい。

.. _Libraries providing Config-file packages:

Config ファイル
---------------

サードパーティが、CMake で使用される実行形式やライブラリ、ヘッダ、その他のファイルを提供する方法として推奨されるのが「:ref:`Config ファイル <Config File Packages>`」です。
これらのファイルはパッケージに同梱されているテキスト・ファイルで、CMake でビルドするターゲット、CMake で参照できる変数、そして CMake コマンドなどを定義します。
Config ファイルは普通の CMake スクリプトで、:command:`find_package` コマンドによって読み込まれまれます。

通常 Config ファイルは ``lib/cmake/<PackageName>`` のパタンに従ったディレクトリの中にありますが、別のディレクトリにある場合もあります（:ref:`search procedure` も参照して下さい）。
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
このファイルは、:command:`find_package` コマンドの呼び出しでパッケージのバージョンが指定された際に、バージョンによる制約を満足するかどうかをチェックする際に使用します。
なお、``<PackageName>ConfigVersion.cmake``  が存在していても、:command:`find_package` コマンドにオプションとして任意のバージョンを渡すことができます。

If the ``<PackageName>Config.cmake`` file is found and any version constraint is satisfied, the :command:`find_package` command considers the package to be found, and the entire package is assumed to be complete as designed.

There may be additional files providing CMake commands or :ref:`imported targets` for you to use.
CMake does not enforce any naming convention for these files.
They are related to the primary ``<PackageName>Config.cmake`` file by use of the CMake :command:`include` command.
The ``<PackageName>Config.cmake`` file would typically include these for you, so they won't usually require any additional step other than the call to :command:`find_package`.

If the location of the package is in a :ref:`directory known to CMake <search procedure>`, the :command:`find_package` call should succeed.
The directories known to CMake are platform-specific.
For example, packages installed on Linux with a standard system package manager will be found in the ``/usr`` prefix automatically.
Packages installed in ``Program Files`` on Windows will similarly be found automatically.

Packages will not be found automatically without help if they are in locations not known to CMake, such as ``/opt/mylib`` or ``$HOME/dev/prefix``.
This is a normal situation, and CMake provides several ways for users to specify where to find such libraries.

The :variable:`CMAKE_PREFIX_PATH` variable may be :ref:`set when invoking CMake <Setting Build Variables>`.
It is treated as a list of base paths in which to search for :ref:`config files <Config File Packages>`.
A package installed in ``/opt/somepackage`` will typically install config files such as ``/opt/somepackage/lib/cmake/somePackage/SomePackageConfig.cmake``.
In that case, ``/opt/somepackage`` should be added to :variable:`CMAKE_PREFIX_PATH`.

The environment variable ``CMAKE_PREFIX_PATH`` may also be populated with prefixes to search for packages.
Like the ``PATH`` environment variable, this is a list, but it needs to use the platform-specific environment variable list item separator (``:`` on Unix and ``;`` on Windows).

The :variable:`CMAKE_PREFIX_PATH` variable provides convenience in cases where multiple prefixes need to be specified, or when multiple packages are available under the same prefix.
Paths to packages may also be specified by setting variables matching ``<PackageName>_DIR``, such as ``SomePackage_DIR``.
Note that this is not a prefix, but should be a full path to a directory containing a config-style package file, such as ``/opt/somepackage/lib/cmake/SomePackage`` in the above example.
See the :command:`find_package` documentation for other CMake variables and environment variables that can affect the search.

.. _Libraries not Providing Config-file Packages:

Find Module Files
-----------------

Packages which do not provide config files can still be found with the
:command:`find_package` command, if a ``FindSomePackage.cmake`` file is
available.  These Find module files are different to config files in that:

#. Find module files should not be provided by the package itself.
#. The availability of a ``Find<PackageName>.cmake`` file does not indicate
   the availability of the package, or any particular part of the package.
#. CMake does not search the locations specified in the
   :variable:`CMAKE_PREFIX_PATH` variable for ``Find<PackageName>.cmake``
   files.  Instead, CMake searches for such files in the locations given
   by the :variable:`CMAKE_MODULE_PATH` variable.  It is common for users to
   set the :variable:`CMAKE_MODULE_PATH` when running CMake, and it is common
   for CMake projects to append to :variable:`CMAKE_MODULE_PATH` to allow use
   of local Find module files.
#. CMake ships ``Find<PackageName>.cmake`` files for some
   :manual:`third party packages <cmake-modules(7)>`.  These files are a
   maintenance burden for CMake, and it is not unusual for these to fall
   behind the latest releases of the packages they are associated with.
   In general, new Find modules are not added to CMake any more.  Projects
   should encourage the upstream packages to provide a config file where
   possible.  If that is unsuccessful, the project should provide its own
   Find module for the package.

See :ref:`Find Modules` for a detailed discussion of how to write a
Find module file.

.. _Imported Targets from Packages:

Imported Targets
----------------

Both config files and Find module files can define :ref:`Imported targets`.
These will typically have names of the form ``SomePrefix::ThingName``.
Where these are available, the project should prefer to use them instead of
any CMake variables that may also be provided.  Such targets typically carry
usage requirements and apply things like header search paths, compiler
definitions, etc. automatically to other targets that link to them (e.g. using
:command:`target_link_libraries`).  This is both more robust and more
convenient than trying to apply the same things manually using variables.
Check the documentation for the package or Find module to see what imported
targets it defines, if any.

Imported targets should also encapsulate any configuration-specific paths.
This includes the location of binaries (libraries, executables), compiler
flags, and any other configuration-dependent quantities.  Find modules may
be less reliable in providing these details than config files.

A complete example which finds a third party package and uses a library
from it might look like the following:

.. code-block:: cmake

  cmake_minimum_required(VERSION 3.10)
  project(MyExeProject VERSION 1.0.0)

  # Make project-provided Find modules available
  list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

  find_package(SomePackage REQUIRED)
  add_executable(MyExe main.cpp)
  target_link_libraries(MyExe PRIVATE SomePrefix::LibName)

Note that the above call to :command:`find_package` could be resolved by
a config file or a Find module.  It uses only the basic arguments supported
by the :ref:`basic signature`.  A ``FindSomePackage.cmake`` file in the
``${CMAKE_CURRENT_SOURCE_DIR}/cmake`` directory would allow the
:command:`find_package` command to succeed using module mode, for example.
If no such module file is present, the system would be searched for a config
file.


Downloading And Building From Source With ``FetchContent``
==========================================================

Dependencies do not necessarily have to be pre-built in order to use them
with CMake.  They can be built from sources as part of the main project.
The :module:`FetchContent` module provides functionality to download
content (typically sources, but can be anything) and add it to the main
project if the dependency also uses CMake.  The dependency's sources will
be built along with the rest of the project, just as though the sources were
part of the project's own sources.

The general pattern is that the project should first declare all the
dependencies it wants to use, then ask for them to be made available.
The following demonstrates the principle (see :ref:`fetch-content-examples`
for more):

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

Various download methods are supported, including downloading and extracting
archives from a URL (a range of archive formats are supported), and a number
of repository formats including Git, Subversion, and Mercurial.
Custom download, update, and patch commands can also be used to support
arbitrary use cases.

When a dependency is added to the project with :module:`FetchContent`, the
project links to the dependency's targets just like any other target from the
project.  If the dependency provides namespaced targets of the form
``SomePrefix::ThingName``, the project should link to those rather than to
any non-namespaced targets.  See the next section for why this is recommended.

Not all dependencies can be brought into the project this way.  Some
dependencies define targets whose names clash with other targets from the
project or other dependencies.  Concrete executable and library targets
created by :command:`add_executable` and :command:`add_library` are global,
so each one must be unique across the whole build.  If a dependency would
add a clashing target name, it cannot be brought directly into the build
with this method.

``FetchContent`` And ``find_package()`` Integration
===================================================

.. versionadded:: 3.24

Some dependencies support being added by either :command:`find_package` or
:module:`FetchContent`.  Such dependencies must ensure they define the same
namespaced targets in both installed and built-from-source scenarios.
A consuming project then links to those namespaced targets and can handle
both scenarios transparently, as long as the project does not use anything
else that isn't provided by both methods.

The project can indicate it is happy to accept a dependency by either method
using the ``FIND_PACKAGE_ARGS`` option to :command:`FetchContent_Declare`.
This allows :command:`FetchContent_MakeAvailable` to try satisfying the
dependency with a call to :command:`find_package` first, using the arguments
after the ``FIND_PACKAGE_ARGS`` keyword, if any.  If that doesn't find the
dependency, it is built from source as described previously instead.

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

The above example calls
:command:`find_package(googletest NAMES GTest) <find_package>` first.
CMake provides a :module:`FindGTest` module, so if that finds a GTest package
installed somewhere, it will make it available, and the dependency will not be
built from source.  If no GTest package is found, it *will* be built from
source.  In either case, the ``GTest::gtest_main`` target is expected to be
defined, so we link our unit test executable to that target.

High-level control is also available through the
:variable:`FETCHCONTENT_TRY_FIND_PACKAGE_MODE` variable.  This can be set to
``NEVER`` to disable all redirection to :command:`find_package`.  It can be
set to ``ALWAYS`` to try :command:`find_package` even if ``FIND_PACKAGE_ARGS``
was not specified (this should be used with caution).

The project might also decide that a particular dependency must be built from
source.  This might be needed if a patched or unreleased version of the
dependency is required, or to satisfy some policy that requires all
dependencies to be built from source.  The project can enforce this by adding
the ``OVERRIDE_FIND_PACKAGE`` keyword to :command:`FetchContent_Declare`.
A call to :command:`find_package` for that dependency will then be redirected
to :command:`FetchContent_MakeAvailable` instead.

.. code-block:: cmake

  include(FetchContent)
  FetchContent_Declare(
    Catch2
    URL https://intranet.mycomp.com/vendored/Catch2_2.13.4_patched.tgz
    URL_HASH MD5=abc123...
    OVERRIDE_FIND_PACKAGE
  )

  # The following is automatically redirected to FetchContent_MakeAvailable(Catch2)
  find_package(Catch2)

For more advanced use cases, see the
:variable:`CMAKE_FIND_PACKAGE_REDIRECTS_DIR` variable.

.. _dependency_providers_overview:

Dependency Providers
====================

.. versionadded:: 3.24

The preceding section discussed techniques that projects can use to specify
their dependencies.  Ideally, the project shouldn't really care where a
dependency comes from, as long as it provides the things it expects (often
just some imported targets).  The project says what it needs and may also
specify where to get it from, in the absence of any other details, so that it
can still be built out-of-the-box.

The developer, on the other hand, may be much more interested in controlling
*how* a dependency is provided to the project.  You might want to use a
particular version of a package that you built yourself.  You might want
to use a third party package manager.  You might want to redirect some
requests to a different URL on a system you control for security or
performance reasons.  CMake supports these sort of scenarios through
:ref:`dependency_providers`.

A dependency provider can be set to intercept :command:`find_package` and
:command:`FetchContent_MakeAvailable` calls.  The provider is given an
opportunity to satisfy such requests before falling back to the built-in
implementation if the provider doesn't fulfill it.

Only one dependency provider can be set, and it can only be set at a very
specific point early in the CMake run.
The :variable:`CMAKE_PROJECT_TOP_LEVEL_INCLUDES` variable lists CMake files
that will be read while processing the first :command:`project()` call (and
only that call).  This is the only time a dependency provider may be set.
At most, one single provider is expected to be used throughout the whole
project.

For some scenarios, the user wouldn't need to know the details of how the
dependency provider is set.  A third party may provide a file that can be
added to :variable:`CMAKE_PROJECT_TOP_LEVEL_INCLUDES`, which will set up
the dependency provider on the user's behalf.  This is the recommended
approach for package managers.  The developer can use such a file like so::

  cmake -DCMAKE_PROJECT_TOP_LEVEL_INCLUDES=/path/to/package_manager/setup.cmake ...

For details on how to implement your own custom dependency provider, see the
:command:`cmake_language(SET_DEPENDENCY_PROVIDER)` command.
