add_library
-----------

.. only:: html

   .. contents::

ソース・ファイルを指定して、ライブラリをプロジェクトに追加する。

通常のライブラリ
^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_library(<name> [STATIC | SHARED | MODULE]
              [EXCLUDE_FROM_ALL]
              [<source>...])

ソース・ファイル ``<source>...`` の :ref:`リスト <CMake Language Lists>` からビルドする ``<name>`` というライブラリをターゲットとして追加します。
``<name>`` はターゲットの論理的な名前であり、プロジェクト内で重複しないグローバルな名前にして下さい。
ターゲットとしてビルドされる実際のファイル名は、ランタイムのプラットフォームに基づいて付与されます（たとえば ``lib<name>.a`` とか ``<name>.lib`` とか）。

.. versionadded:: 3.1
  このコマンドに渡す ``<source>...`` に ``$<...>`` のような :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を指定できるようになった。
  利用可能な式について詳細は :manual:`cmake-generator-expressions(7)` を参照のこと。

.. versionadded:: 3.11
  あとで :command:`target_sources` コマンドで ``<source>...`` を追加する場合、これらの引数を省略できるようになった。

ビルドするライブラリの種類は ``STATIC``、``SHARED``、または ``MODULE`` で指定します。
``STATIC`` なライブラリは、他のターゲットにリンクする際に使用するオブジェクトのアーカイブです。
``SHARED`` なライブラリは、実行時に動的にリンクされてロードされます。
``MODULE`` なライブラリは、他のターゲットにはリンクされないプラグインと呼ばれるもので、``dlopen()`` のような関数を使って実行時に動的にロードされます。
ライブラリの種類を明示的に指定しない場合は、:variable:`BUILD_SHARED_LIBS` 変数が ``ON`` かどうかに応じて CMake が ``STATIC`` または ``SHARED`` を選択します。
``SHARED`` と ``MODULE`` のライブラリの場合、自動的に :prop_tgt:`POSITION_INDEPENDENT_CODE` というターゲット・プロパティに ``ON`` がセットされます。
``SHARED`` ライブラリの場合、macOS 系向けのフレームワークを生成するために :prop_tgt:`FRAMEWORK` というターゲット・プロパティが付与される場合があります。

.. versionadded:: 3.8
  ``STATIC`` ライブラリの場合、静的フレームワークを生成するために :prop_tgt:`FRAMEWORK` というターゲット・プロパティが付与されるようになった。

まったくシンボルを外部に公開していないライブラリを ``SHARED`` として指定しないで下さい。
たとえば、アンマネージドなシンボルを外部に公開していない Windows 系のリソース DLL や C++/CLI の DLL は ``SHARED`` ではなく ``MODULE`` を指定して下さい。
これは、CMake が Windows 上で ``SHARED`` に関連づけられたインポート・ライブラリ（たとえば、``.lib``）が常に存在しているものと想定しているためです。

デフォルトでライブラリのファイルは、このコマンドを呼び出したソースツリーに対応したビルドツリーに相当するディレクトリに作成されます。
この場所を変更する方法については :prop_tgt:`ARCHIVE_OUTPUT_DIRECTORY` や :prop_tgt:`LIBRARY_OUTPUT_DIRECTORY` や :prop_tgt:`RUNTIME_OUTPUT_DIRECTORY` というターゲット・プロパティのドキュメントを参照して下さい。
``<name>`` を、最終的にビルドされるファイル名に変更する方法については :prop_tgt:`OUTPUT_NAME` というターゲット・プロパティのドキュメントを参照して下さい。

``EXCLUDE_FROM_ALL`` オプションを指定すると、対応するプロパティがビルドしたターゲットに付与されます。
詳細は :prop_tgt:`EXCLUDE_FROM_ALL` というターゲット・プロパティを参照して下さい。

ビルドシステムのプロパティ定義について詳細は :manual:`cmake-buildsystem(7)` を参照して下さい。

ソース・ファイルの一部が前処理されて変更されている時に、IDE から処理する前のソース・ファイルにアクセスできるようにする方法については :prop_sf:`HEADER_FILE_ONLY` というプロパティも参照して下さい。

オブジェクト・ライブラリ
^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_library(<name> OBJECT [<source>...])

:ref:`オブジェクト・ライブラリ <Object Libraries>` をターゲットとして追加します。
これはソース・ファイルをコンパイルするだけで、そこで生成されたオブジェクト・ファイルをアーカイブにしたり、他のライブラリにリンクしたりすることはありません。
この ``add_library`` や :command:`add_executable` コマンドでビルドした別のターゲットは、:genex:`$\<TARGET_OBJECTS:objlib\> <TARGET_OBJECTS>` のジェネレータ式（``objlib`` はオブジェクト・ライブラリの名前）を利用して、オブジェクト・ファイルをソース・ファイルの一部として参照できます。
たとえば、次のコマンドを実行すると：

.. code-block:: cmake

  add_library(... $<TARGET_OBJECTS:objlib> ...)
  add_executable(... $<TARGET_OBJECTS:objlib> ...)

``objlib`` というオブジェクト・ライブラリが、別のソースからコンパイルされる実行形式のオブジェクト・ファイルに含まれます。
生成されるオブジェクト・ライブラリには、コンパイルするソース・ファイル、ヘッダ・ファイル、そして通常のライブラリとしてリンクには影響を与えないその他のファイル（例えば ``.txt``）だけが含まれます。
これらには、:ref:`add_custom_command <add_custom_command(TARGET)>` コマンドでそのようなソースを生成する独自のコマンドが含まれている場合がありますが、``PRE_BUILD`` や ``PRE_LINK`` や ``POST_BUILD`` が指定されたコマンドは含まれません。
ただし、オブジェクト・ファイルしか持たないターゲットを好まない Xcode のような一部のターゲットのビルドシステムでは、 :genex:`$\<TARGET_OBJECTS:objlib\> <TARGET_OBJECTS>` のジェネレータ式を参照するターゲットに、少なくとも1個の実ソース・ファイルを追加することを検討してみて下さい。

.. versionadded:: 3.12
  :command:`target_link_libraries` コマンドでオブジェクト・ライブラリをリンクできるようになった。

INTERFACE ライブラリ
^^^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_library(<name> INTERFACE)

:ref:`INTERFACE ライブラリ <Interface Libraries>` をターゲットとして追加します。
このターゲットはソース・ファイルをコンパイルしないので、ライブラリに相当するファイルは生成されません。
ただしターゲット・プロパティを設定することは可能であり、プロパティをインストールしたりエキスポートすることが可能です。
通常、``INTERFACE_*`` 系のプロパティは、次のコマンドを使ってターゲットである ``INTERFACE`` 型のライブラリに付与されます：

* :command:`set_property`
* :command:`target_link_libraries(INTERFACE)`
* :command:`target_link_options(INTERFACE)`
* :command:`target_include_directories(INTERFACE)`
* :command:`target_compile_options(INTERFACE)`
* :command:`target_compile_definitions(INTERFACE)`
* :command:`target_sources(INTERFACE)`

さらに、他のターゲットと同様に、:command:`target_link_libraries` コマンドの引数として指定できます。

この ``add_library(INTERFACE)`` コマンドで生成された ``INTERFACE`` 型のライブラリには、それ自体にソース・ファイルは無く、ビルドシステム内ではターゲットとして扱われません。

.. versionadded:: 3.15
  ``INTERFACE`` 型のライブラリに :prop_tgt:`PUBLIC_HEADER` と :prop_tgt:`PRIVATE_HEADER` というターゲット・プロパティを付与できるようになった。
  これらのプロパティで指定されたヘッダ・ファイルを :command:`install(TARGETS)` コマンドでインストールできる。

.. versionadded:: 3.19
  ``INTERFACE`` 型のライブラリにソース・ファイルを指定できるようになった：

  .. code-block:: cmake

    add_library(<name> INTERFACE [<source>...] [EXCLUDE_FROM_ALL])

  ソース・ファイル ``<source>...`` の :ref:`リスト <CMake Language Lists>` を引数としてそのまま ``add_library`` コマンドに渡すか、または ``PRIVATE`` や ``PUBLIC`` オプション付きで :command:`target_sources` コマンドを呼び出して、``add_library`` コマンドのあとからソース・ファイルを追加できる。

  ターゲットがソース・ファイル（:prop_tgt:`SOURCES` というターゲット・プロパティが付与されたファイル）やヘッダ・ファイル（:prop_tgt:`HEADER_SETS` というターゲット・プロパティが付与されたファイル）を持つ ``INTERFACE`` 型のライブラリの場合、ビルドシステムの中でビルド・ターゲットとして扱われるようになる（すなわち :command:`add_custom_target` コマンドで定義したターゲットと同じ扱い）。
  ただし、この場合でもソース・ファイルのコンパイルは行わないが、:command:`add_custom_command` コマンドで定義した独自コマンドのビルド・ルールは含まれる。
                  

.. note::
  ``INTERFACE`` オプションを指定できる大部分のコマンドで、このオプションのあとに :ref:`リスト <CMake Language Lists>` する引数はターゲットの「:ref:`利用要件 <Target Usage Requirements>`」（*Usage Requirements*）に追加されるだけであり、ターゲットのソースではない。
  そして、この ``add_library(INTERFACE)`` コマンドの ``INTERFACE`` オプションはあくまでもライブラリの種類だけを参照するものである。
  そのため、このオプションに渡す :ref:`リスト <CMake Language Lists>` された ``<source>...`` は ``INTERFACE`` 型のライブラリに対して ``PRIVATE`` な扱いであり、ターゲット・プロパティの :prop_tgt:`INTERFACE_SOURCES` には含まれない。

.. _`add_library imported libraries`:

IMPORTED なライブラリ
^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_library(<name> <type> IMPORTED [GLOBAL])

``<name>`` という名前を持つ :ref:`IMPORTED なライブラリ <Imported Targets>` をターゲットとして追加します。
このライブラリをビルドするためのルールは生成されず、:prop_tgt:`IMPORTED` というターゲット・プロパティを ``True`` にセットするだけです。
``<name>`` のスコープは、このライブラリをビルドしたディレクトリとそのサブディレクトリですが、``GLOBAL`` オプションを指定するとプロジェクト全体に拡張されます
（つまり、プロジェクト内の全てのビルド・ターゲットから参照できます）。
この ``IMPORTED`` なライブラリは  :command:`target_link_libraries` などのコマンドからの参照に便利です。
このライブラリの詳細は ``IMPORTED_`` や ``INTERFACE_`` で始まる名前のプロパティを設定することで指定できます。

``<type>`` には次のいずれかを指定して下さい：

``STATIC``, ``SHARED``, ``MODULE``, ``UNKNOWN``
  プロジェクトの外部にあるライブラリ・ファイルを参照する。
  :prop_tgt:`IMPORTED_LOCATION` （またはビルド構成ごとの :prop_tgt:`IMPORTED_LOCATION_<CONFIG>`）というターゲット・プロパティは実際にライブラリ・ファイルがある場所を表す。

  * Windows 系以外のプラットフォームの ``SHARED`` ライブラリはリンカやローダの両方で使用される ``.so`` や ``.dylib`` ファイルである。
    もし参照するライブラリ・ファイルに ``SONAME`` （または MacOS 系のプラットフォームの場合は ``@rpath/`` で始まる ``LC_ID_DYLIB`` ）がある場合は、その内容を :prop_tgt:`IMPORTED_SONAME` というターゲット・プロパティにもセットすること。
    参照するライブラリ・ファイルに ``SONAME`` は無いが、ホストのプラットフォームが ``SONAME`` をサポートしている場合は :prop_tgt:`IMPORTED_NO_SONAME` というターゲット・プロパティをセットすること。

  * Windows 系プラットフォームの ``SHARED`` ライブラリの場合、 :prop_tgt:`IMPORTED_IMPLIB` （またはビルド構成ごとの :prop_tgt:`IMPORTED_IMPLIB_<CONFIG>`）というターゲット・プロパティには実際に DLL インポート・ライブラリのファイル（``.lib`` や ``.dll.a``）がある場所を指定し、 ``IMPORTED_LOCATION`` には実際にランタイム・ライブラリのファイル（``.dll``） がある場所を指定する（後者はオプションであるが、:genex:`TARGET_RUNTIME_DLLS` というジェネレータ式で必要になるので指定することを推奨する）。

  追加する「:ref:`利用要件 <Target Usage Requirements>`」（*Usage Requirements*）は ``INTERFACE_*`` 系のプロパティで指定することも可能である。

  An ``UNKNOWN`` library type is typically only used in the implementation of :ref:`Find Modules`.
  It allows the path to an imported library (often found using the :command:`find_library` command) to be used without having to know what type of library it is.
  This is especially useful on Windows where a static library and a DLL's import library both have the same file extension.

``OBJECT``
  References a set of object files located outside the project.
  The :prop_tgt:`IMPORTED_OBJECTS` target property (or its per-configuration variant :prop_tgt:`IMPORTED_OBJECTS_<CONFIG>`) specifies the locations of object files on disk.
  Additional usage requirements may be specified in ``INTERFACE_*`` properties.

``INTERFACE``
  Does not reference any library or object files on disk, but may specify usage requirements in ``INTERFACE_*`` properties.

さらに詳細は ``IMPORTED_*`` や ``INTERFACE_*`` 系のプロパティのドキュメントをそれぞれ参照して下さい。

ALIAS なライブラリ
^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_library(<name> ALIAS <target>)

Creates an :ref:`Alias Target <Alias Targets>`, such that ``<name>`` can be
used to refer to ``<target>`` in subsequent commands.  The ``<name>`` does
not appear in the generated buildsystem as a make target.  The ``<target>``
may not be an ``ALIAS``.

.. versionadded:: 3.11
  An ``ALIAS`` can target a ``GLOBAL`` :ref:`Imported Target <Imported Targets>`

.. versionadded:: 3.18
  An ``ALIAS`` can target a non-``GLOBAL`` Imported Target. Such alias is
  scoped to the directory in which it is created and below.
  The :prop_tgt:`ALIAS_GLOBAL` target property can be used to check if the
  alias is global or not.

``ALIAS`` targets can be used as linkable targets and as targets to
read properties from.  They can also be tested for existence with the
regular :command:`if(TARGET)` subcommand.  The ``<name>`` may not be used
to modify properties of ``<target>``, that is, it may not be used as the
operand of :command:`set_property`, :command:`set_target_properties`,
:command:`target_link_libraries` etc.  An ``ALIAS`` target may not be
installed or exported.

参考情報
^^^^^^^^

* :command:`add_executable`
