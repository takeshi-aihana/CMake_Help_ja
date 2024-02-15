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

:ref:`リスト <CMake Language Lists>` されたソース・ファイル ``<source>...`` からビルドする ``<name>`` というライブラリをターゲットとして追加します。
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
これはソース・ファイルをコンパイルするだけで、そこで生成されたオブジェクト・ファイルをアーカイブしたり、他のライブラリにリンクしたりすることはありません。
この ``add_library`` や :command:`add_executable` コマンドでビルドした別のターゲットは、:genex:`$\<TARGET_OBJECTS:objlib\> <TARGET_OBJECTS>` のジェネレータ式（``objlib`` はオブジェクト・ライブラリの名前）を利用して、オブジェクト・ファイルをソース・ファイルの一部として参照できます。
たとえば、次のコマンドを実行すると：

.. code-block:: cmake

  add_library(... $<TARGET_OBJECTS:objlib> ...)
  add_executable(... $<TARGET_OBJECTS:objlib> ...)

``objlib`` というオブジェクト・ライブラリが、別のソースからコンパイルされる実行形式のオブジェクト・ファイルに含まれます。
生成されるオブジェクト・ライブラリには、コンパイルするソース・ファイル、ヘッダ・ファイル、そして通常のライブラリとしてリンクには影響を与えないその他のファイル（例えば ``.txt``）だけが含まれます。
これらには、:ref:`add_custom_command <add_custom_command(TARGET)>` コマンドでそのようなソースを生成する独自のコマンドが含まれている場合がありますが、``PRE_BUILD`` や ``PRE_LINK`` や ``POST_BUILD`` が指定されたコマンドは含まれません。
オブジェクト・ファイルしか持たないターゲットを好まない Xcode といった一部のターゲットのビルドシステムでは、 :genex:`$\<TARGET_OBJECTS:objlib\> <TARGET_OBJECTS>` のジェネレータ式を参照するターゲットに、少なくとも1個の実ソース・ファイルを追加することを検討してみて下さい。

.. versionadded:: 3.12
  :command:`target_link_libraries` コマンドでオブジェクト・ライブラリをリンクできるようになった。

Interface Libraries
^^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_library(<name> INTERFACE)

Creates an :ref:`Interface Library <Interface Libraries>`.
An ``INTERFACE`` library target does not compile sources and does
not produce a library artifact on disk.  However, it may have
properties set on it and it may be installed and exported.
Typically, ``INTERFACE_*`` properties are populated on an interface
target using the commands:

* :command:`set_property`,
* :command:`target_link_libraries(INTERFACE)`,
* :command:`target_link_options(INTERFACE)`,
* :command:`target_include_directories(INTERFACE)`,
* :command:`target_compile_options(INTERFACE)`,
* :command:`target_compile_definitions(INTERFACE)`, and
* :command:`target_sources(INTERFACE)`,

and then it is used as an argument to :command:`target_link_libraries`
like any other target.

An interface library created with the above signature has no source files
itself and is not included as a target in the generated buildsystem.

.. versionadded:: 3.15
  An interface library can have :prop_tgt:`PUBLIC_HEADER` and
  :prop_tgt:`PRIVATE_HEADER` properties.  The headers specified by those
  properties can be installed using the :command:`install(TARGETS)` command.

.. versionadded:: 3.19
  An interface library target may be created with source files:

  .. code-block:: cmake

    add_library(<name> INTERFACE [<source>...] [EXCLUDE_FROM_ALL])

  Source files may be listed directly in the ``add_library`` call or added
  later by calls to :command:`target_sources` with the ``PRIVATE`` or
  ``PUBLIC`` keywords.

  If an interface library has source files (i.e. the :prop_tgt:`SOURCES`
  target property is set), or header sets (i.e. the :prop_tgt:`HEADER_SETS`
  target property is set), it will appear in the generated buildsystem
  as a build target much like a target defined by the
  :command:`add_custom_target` command.  It does not compile any sources,
  but does contain build rules for custom commands created by the
  :command:`add_custom_command` command.

.. note::
  In most command signatures where the ``INTERFACE`` keyword appears,
  the items listed after it only become part of that target's usage
  requirements and are not part of the target's own settings.  However,
  in this signature of ``add_library``, the ``INTERFACE`` keyword refers
  to the library type only.  Sources listed after it in the ``add_library``
  call are ``PRIVATE`` to the interface library and do not appear in its
  :prop_tgt:`INTERFACE_SOURCES` target property.

.. _`add_library imported libraries`:

Imported Libraries
^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_library(<name> <type> IMPORTED [GLOBAL])

Creates an :ref:`IMPORTED library target <Imported Targets>` called ``<name>``.
No rules are generated to build it, and the :prop_tgt:`IMPORTED` target
property is ``True``.  The target name has scope in the directory in which
it is created and below, but the ``GLOBAL`` option extends visibility.
It may be referenced like any target built within the project.
``IMPORTED`` libraries are useful for convenient reference from commands
like :command:`target_link_libraries`.  Details about the imported library
are specified by setting properties whose names begin in ``IMPORTED_`` and
``INTERFACE_``.

The ``<type>`` must be one of:

``STATIC``, ``SHARED``, ``MODULE``, ``UNKNOWN``
  References a library file located outside the project.  The
  :prop_tgt:`IMPORTED_LOCATION` target property (or its per-configuration
  variant :prop_tgt:`IMPORTED_LOCATION_<CONFIG>`) specifies the
  location of the main library file on disk:

  * For a ``SHARED`` library on most non-Windows platforms, the main library
    file is the ``.so`` or ``.dylib`` file used by both linkers and dynamic
    loaders.  If the referenced library file has a ``SONAME`` (or on macOS,
    has a ``LC_ID_DYLIB`` starting in ``@rpath/``), the value of that field
    should be set in the :prop_tgt:`IMPORTED_SONAME` target property.
    If the referenced library file does not have a ``SONAME``, but the
    platform supports it, then  the :prop_tgt:`IMPORTED_NO_SONAME` target
    property should be set.

  * For a ``SHARED`` library on Windows, the :prop_tgt:`IMPORTED_IMPLIB`
    target property (or its per-configuration variant
    :prop_tgt:`IMPORTED_IMPLIB_<CONFIG>`) specifies the location of the
    DLL import library file (``.lib`` or ``.dll.a``) on disk, and the
    ``IMPORTED_LOCATION`` is the location of the ``.dll`` runtime
    library (and is optional, but needed by the :genex:`TARGET_RUNTIME_DLLS`
    generator expression).

  Additional usage requirements may be specified in ``INTERFACE_*`` properties.

  An ``UNKNOWN`` library type is typically only used in the implementation of
  :ref:`Find Modules`.  It allows the path to an imported library (often found
  using the :command:`find_library` command) to be used without having to know
  what type of library it is.  This is especially useful on Windows where a
  static library and a DLL's import library both have the same file extension.

``OBJECT``
  References a set of object files located outside the project.
  The :prop_tgt:`IMPORTED_OBJECTS` target property (or its per-configuration
  variant :prop_tgt:`IMPORTED_OBJECTS_<CONFIG>`) specifies the locations of
  object files on disk.
  Additional usage requirements may be specified in ``INTERFACE_*`` properties.

``INTERFACE``
  Does not reference any library or object files on disk, but may
  specify usage requirements in ``INTERFACE_*`` properties.

See documentation of the ``IMPORTED_*`` and ``INTERFACE_*`` properties
for more information.

Alias Libraries
^^^^^^^^^^^^^^^

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
