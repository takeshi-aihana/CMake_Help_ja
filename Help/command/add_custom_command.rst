add_custom_command
------------------

ビルドシステムに独自のビルド・ルールを追加する。

この ``add_custom_command`` コマンドの目的は二つあります。

ファイルを生成する
^^^^^^^^^^^^^^^^^^

CMake 実行時に「新たなファイルを出力する」コマンドを追加します：

.. code-block:: cmake

  add_custom_command(OUTPUT output1 [output2 ...]
                     COMMAND command1 [ARGS] [args1...]
                     [COMMAND command2 [ARGS] [args2...] ...]
                     [MAIN_DEPENDENCY depend]
                     [DEPENDS [depends...]]
                     [BYPRODUCTS [files...]]
                     [IMPLICIT_DEPENDS <lang1> depend1
                                      [<lang2> depend2] ...]
                     [WORKING_DIRECTORY dir]
                     [COMMENT comment]
                     [DEPFILE depfile]
                     [JOB_POOL job_pool]
                     [JOB_SERVER_AWARE <bool>]
                     [VERBATIM] [APPEND] [USES_TERMINAL]
                     [COMMAND_EXPAND_LISTS]
                     [DEPENDS_EXPLICIT_ONLY])

これは ``OUTPUT`` オプションに指定したファイルを、ビルド時に出力する独自コマンド ``COMMAND`` を定義します。
この ``add_custom_command`` コマンドを呼び出している ``CMakeLists.txt`` と同じディレクトリに生成されるターゲットに、ファイルを出力するためのルールが追加されます。

並列ビルドにより、このルールが他と衝突する可能性があるので、独立した複数のターゲットで同じ ``OUTPUT`` を指定しないようにして下さい。
そのような場合は :command:`add_custom_target` コマンドを使って、複数のターゲットの間で依存関係を明示して、個別に出力するにようにして下さい。
詳細は `Example: Generating Files for Multiple Targets`_ のサンプルを参照して下さい。

このモードのオプションは次のとおりです：

``APPEND``
  最初の出力コマンドに ``COMMAND`` と ``DEPENDS`` オプションの値を追加する。
  このコマンドを事前に呼び出しておいて、既に出力先のファイルが存在している必要がある。

  以前の呼び出しで「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使って出力先を指定していた場合は、今回の呼び出しでも同じ出力先にすること。
  この場合、定義した独自コマンドと依存関係は全てのビルド構成に適用される。

  このオプションを指定した場合、現在は ``COMMENT``、``MAIN_DEPENDENCY``、``WORKING_DIRECTORY`` のオプションを無視する（将来的に変更される可能性あり）。

``BYPRODUCTS``
  .. versionadded:: 3.2

  この ``add_custom_command`` コマンドによって出力させるファイル ``files ...`` を指定する。
  ただし、それらの変更時刻が、依存関係によって新しい場合と古い場合とでバラバラになる場合がある。
  このオプションで指定した ``files ...`` が相対パスの場合は、:variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとして絶対パスを計算する。
  この ``files ...`` には :prop_sf:`GENERATED` というソース・ファイルのプロパティが自動的に付与される。

  *この機能の背景については* :policy:`CMP0058` *ポリシーを参照して下さい* 。

  このオプションによる ``files ...`` の明示的な指定は :generator:`Ninja` ジェネレータでサポートされており、``files ...`` が存在していない場合の生成方法をジェネレータに指示する。
  他のビルド・ルールが ``files ...`` に依存している場合に便利である。
  Ninja ジェネレータは、依存ファイルがビルドされる前に ``files ...`` を確実に参照できるようにするために、順序のみの依存関係であって、別のルールが依存しているファイルを出力するためのルールが必要である（FIXME: 意味不明）。

  :ref:`Makefile Generators` の場合は ``make clean`` 時に ``BYPRODUCTS`` と :prop_sf:`GENERATED` プロパティ付きのファイルを削除する。

  .. versionadded:: 3.20
    ``BYPRODUCTS`` オプションの引数に、制限された一連の「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使用できるようになった。
    ただし :ref:`Target-dependent expressions <Target-Dependent Queries>` は利用できない。

``COMMAND``
  ターゲットのビルド時に実行するコマンドライン（``command1`` や ``command2``）を指定する。
  ``COMMAND`` オプションを複数指定すると順番に実行していくが、シェル・スクリプトやバッチ・スクリプトの類に *再構成しているわけではない*
  （完全なスクリプトとして実行する場合は :command:`configure_file` コマンドや :command:`file(GENERATE)` コマンドを使用して、実際にスクリプトを作成し、それを ``COMMAND`` オプションで起動すること）。
  サブオプションの ``ARGS`` は下位互換性のためのもので、指定しても無視される。

  ``COMMAND`` オプションに、:command:`add_executable` コマンドで追加したターゲット名を指定した場合、次のいずれかの条件に該当する時は、そのターゲット名が実際にビルドされた実行形式のパス名に置き換えられる：

  * そのターゲットがクロス・コンパイルされたものではない（すなわち CMake 変数の :variable:`CMAKE_CROSSCOMPILING` が true ではない）。
  * .. versionadded:: 3.6
      そのターゲットはクロス・コンパイルされているが、それを実行するためのエミュレータが提供されている（すなわち :prop_tgt:`CROSSCOMPILING_EMULATOR` というターゲットのプロパティが付与されている）。
      この場合、ターゲットのパス名の前に :prop_tgt:`CROSSCOMPILING_EMULATOR` プロパティの内容（エミュレータ）が自動的に追加される。

  上記の条件のどちらにも該当しない場合、ビルド時に ``PATH`` 内で見つかった同名の実行形式であると想定する。

  ``COMMAND`` オプションに渡す引数には :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を指定できる。
  たとえば :genex:`TARGET_FILE` というジェネレータ式を使うと、コマンドラインの後半で（コマンドの引数として）ターゲットのパスを参照できる。

  ジェネレータ式から得られた次のターゲットのいずれかをコマンドとして指定するか、またはコマンドの引数として指定すると、自動的に「ターゲット・レベル」の依存関係を追加し、コマンドラインを実行する前に、追加した依存先の実行形式を先にビルドする（:policy:`CMP0112` ポリシーも参照のこと）：

    * ``TARGET_FILE``
    * ``TARGET_LINKER_FILE``
    * ``TARGET_SONAME_FILE``
    * ``TARGET_PDB_FILE``

  この依存関係により、依存先の実行形式が再コンパイルされるたびに、``COMMAND`` のコマンドラインが実行されることはない。
  逆に、コマンドラインを実行させたい場合は、``DEPENDS`` オプションに依存するターゲットを追加しておくこと。


``COMMENT``
  ビルド時にコマンドラインを実行する前に、指定したメッセージを出力する。

  .. versionadded:: 3.26
    ``COMMENT`` オプションに渡す引数に :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を指定できるようになった。

``DEPENDS``
  ``COMMNAND`` に指定したコマンドラインの実行に依存するファイルを指定する。
  ``depends ...`` のファイルはそれぞれ、次の条件ごとに依存関係を作成する：

  1. このオプションの引数が、:command:`add_custom_target` や :command:`add_executable` あるいは :command:`add_library` といったコマンドで生成したファイルの場合は「ターゲット・レベル」の依存関係を作成し、この ``add_custom_command`` を呼び出すターゲットよりも前に、これらのファイルをビルドする。
     さらに、それらのファイルが実行形式またはライブラリの場合、この ``add_custom_command`` コマンドを呼び出すターゲットが再コンパイルされて ``COMMAND`` が実行されるように「ファイル・レベル」の依存関係を作成する。

  2. このオプションの引数が絶対パスの場合、そのパス上に「ファイル・レベル」の依存関係を作成する。

  3. このオプションの引数が、この ``add_custom_command`` を呼び出すターゲットに追加されたソース・ファイル、または :ref:`ソース・ファイルのプロパティ <Source File Properties>` が付与されているソース・ファイルの場合、そのファイルに対して「ファイル・レベル」の依存関係を作成する。

  4. このオプションの引数が相対パスで、:variable:`CMAKE_CURRENT_SOURCE_DIR` に存在している場合、:variable:`CMAKE_CURRENT_SOURCE_DIR` 内のそのファイルに対して「ファイル・レベル」の依存関係を作成する。

  5. 以外の条件を満足しない場合、:variable:`CMAKE_CURRENT_BINARY_DIR` をベースディレクトリとした相対パス上に「ファイル・レベル」の依存関係を作成する。

  すべての依存関係が ``CMakeLists.txt`` と同じディレクトリの別の ``add_custom_command`` の出力である場合、CMake はこの ``add_custom_command`` を呼び出すターゲットに、別の ``add_custom_command`` を自動的に取り込む。

  .. versionadded:: 3.16
    依存関係が同じディレクトリ内のターゲットまたはそのビルド イベントの ``BYPRODUCTS`` として引数に指定されている場合、そこで生成するファイルを確実に利用できるようにするために「ターゲット・レベル」の依存関係を作成するようになった。

  ``DEPENDS`` オプションを指定しない場合、``COMMAND`` のコマンドラインは ``OUTPUT`` が存在していない時にだけ実行される
  （つまり ``COMMAND`` のコマンドラインが ``OUTPUT`` を生成しないと、常にそのコマンドラインが実行されることになる）。

  .. versionadded:: 3.1
    ``DEPENDS`` オプションに渡す引数に :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を指定できるようになった。

``COMMAND_EXPAND_LISTS``
  .. versionadded:: 3.8

  ``COMMAND`` オプションに指定したコマンドラインの文字列を、:manual:`ジェネレータ式 <cmake-generator-expressions(7)>` も含めすべて展開する。
  これにより、たとえば ``${CC} "-I$<JOIN:$<TARGET_PROPERTY:foo,INCLUDE_DIRECTORIES>,;-I>" foo.cc`` のようなコマンドラインを適切に実行することができる。

``IMPLICIT_DEPENDS``
  Request scanning of implicit dependencies of an input file.
  The language given specifies the programming language whose corresponding dependency scanner should be used.
  Currently only ``C`` and ``CXX`` language scanners are supported.
  The language has to be specified for every file in the ``IMPLICIT_DEPENDS`` list.
  Dependencies discovered from the scanning are added to those of the custom command at build time.
  Note that the ``IMPLICIT_DEPENDS`` option is currently supported only for Makefile generators and will be ignored by other generators.

  .. note::

    このオプションは ``DEPFILE`` オプションと同時には指定できない。

``JOB_POOL``
  .. versionadded:: 3.15

  :generator:`Ninja` ジェネレータ向けに :prop_gbl:`JOB_POOLS` というプロパティを指定する。
  ``USES_TERMINAL`` オプションとは互換性はない。
  :prop_gbl:`JOB_POOLS` のプロパティで定義されていないプールを使用するとビルド時にエラーになる。

``JOB_SERVER_AWARE``
  .. versionadded:: 3.28

  この ``add_custom_command`` で追加したコマンドラインが GNU Make のジョブ・サーバ対応であることを CMake に伝える。

  :generator:`Unix Makefiles`、:generator:`MSYS Makefiles`、:generator:`MinGW Makefiles` のジェネレータを使用すると、レシピ行の先頭に ``+`` が追加される。
  詳細は `GNU Make Documentation`_ を参照のこと。

  このオプションは、他のジェネレータによって暗黙的に無視される。

.. _`GNU Make Documentation`: https://www.gnu.org/software/make/manual/html_node/MAKE-Variable.html

``MAIN_DEPENDENCY``
  Specify the primary input source file to the command.
  This is treated just like any value given to the ``DEPENDS`` option but also suggests to Visual Studio generators where to hang the custom command.
  Each source file may have at most one command specifying it as its main dependency.
  A compile command (i.e. for a library or an executable) counts as an implicit main dependency which gets silently overwritten by a custom command specification.

``OUTPUT``
  ``COMMAND`` のコマンドラインが出力する予定のファイルを指定する。
  出力された ``output1 output2 ...`` にはそれぞれ :prop_sf:`GENERATED` というソース・ファイルのプロパティが自動的に付与される。
  ``COMMAND`` のコマンドラインが、これらのファイルを作成しないことが判明している場合は :prop_sf:`SYMBOLIC` というソース・ファイルのプロパティを付与しておく必要がある。

  相対パスで出力ファイルを指定すると、次をベース・ディレクトリとして絶対パスに変換される：

  1. :variable:`CMAKE_CURRENT_BINARY_DIR` （ビルド・ディレクトリ）または

  2. :variable:`CMAKE_CURRENT_SOURCE_DIR` （ソース・ディレクトリ）

  上記はビルド・ディレクトリが優先される。

  .. versionadded:: 3.20
    ``OUTPUT`` オプションの引数に、制限された一連の「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使用できるようになった。
    ただし :ref:`Target-dependent expressions <Target-Dependent Queries>` は利用できない。

``USES_TERMINAL``
  .. versionadded:: 3.2

  The command will be given direct access to the terminal if possible.
  With the :generator:`Ninja` generator, this places the command in the ``console`` :prop_gbl:`pool <JOB_POOLS>`.

``VERBATIM``
  All arguments to the commands will be escaped properly for the build tool so that the invoked command receives each argument unchanged.
  Note that one level of escapes is still used by the CMake language processor before add_custom_command even sees the arguments.
  Use of ``VERBATIM`` is recommended as it enables correct behavior.
  When ``VERBATIM`` is not given the behavior is platform specific because there is no protection of tool-specific special characters.

``WORKING_DIRECTORY``
  Execute the command with the given current working directory.
  If it is a relative path it will be interpreted relative to the build tree directory corresponding to the current source directory.

  .. versionadded:: 3.13
    Arguments to ``WORKING_DIRECTORY`` may use :manual:`generator expressions <cmake-generator-expressions(7)>`.

``DEPFILE``
  .. versionadded:: 3.7

  Specify a depfile which holds dependencies for the custom command.
  It is usually emitted by the custom command itself.
  This keyword may only be used if the generator supports it, as detailed below.

  The expected format, compatible with what is generated by ``gcc`` with the option ``-M``, is independent of the generator or platform.

  The formal syntax, as specified using `BNF <https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form>`_ notation with the regular extensions, is the following:

  .. raw:: latex

    \begin{small}

  .. productionlist:: depfile
    depfile: `rule`*
    rule: `targets` (':' (`separator` `dependencies`?)?)? `eol`
    targets: `target` (`separator` `target`)* `separator`*
    target: `pathname`
    dependencies: `dependency` (`separator` `dependency`)* `separator`*
    dependency: `pathname`
    separator: (`space` | `line_continue`)+
    line_continue: '\' `eol`
    space: ' ' | '\t'
    pathname: `character`+
    character: `std_character` | `dollar` | `hash` | `whitespace`
    std_character: <any character except '$', '#' or ' '>
    dollar: '$$'
    hash: '\#'
    whitespace: '\ '
    eol: '\r'? '\n'

  .. raw:: latex

    \end{small}

  .. note::

    As part of ``pathname``, any slash and backslash is interpreted as
    a directory separator.

  .. versionadded:: 3.7
    The :generator:`Ninja` generator supports ``DEPFILE`` since the keyword
    was first added.

  .. versionadded:: 3.17
    Added the :generator:`Ninja Multi-Config` generator, which included
    support for the ``DEPFILE`` keyword.

  .. versionadded:: 3.20
    Added support for :ref:`Makefile Generators`.

    .. note::

      ``DEPFILE`` cannot be specified at the same time as the
      ``IMPLICIT_DEPENDS`` option for :ref:`Makefile Generators`.

  .. versionadded:: 3.21
    Added support for :ref:`Visual Studio Generators` with VS 2012 and above,
    and for the :generator:`Xcode` generator.  Support for
    :manual:`generator expressions <cmake-generator-expressions(7)>` was also
    added.

  Using ``DEPFILE`` with generators other than those listed above is an error.

  If the ``DEPFILE`` argument is relative, it should be relative to
  :variable:`CMAKE_CURRENT_BINARY_DIR`, and any relative paths inside the
  ``DEPFILE`` should also be relative to :variable:`CMAKE_CURRENT_BINARY_DIR`.
  See policy :policy:`CMP0116`, which is always ``NEW`` for
  :ref:`Makefile Generators`, :ref:`Visual Studio Generators`,
  and the :generator:`Xcode` generator.

``DEPENDS_EXPLICIT_ONLY``

  .. versionadded:: 3.27

  Indicates that the command's ``DEPENDS`` argument represents all files
  required by the command and implicit dependencies are not required.

  Without this option, if any target uses the output of the custom command,
  CMake will consider that target's dependencies as implicit dependencies for
  the custom command in case this custom command requires files implicitly
  created by those targets.

  This option can be enabled on all custom commands by setting
  :variable:`CMAKE_ADD_CUSTOM_COMMAND_DEPENDS_EXPLICIT_ONLY` to ``ON``.

  Only the :ref:`Ninja Generators` actually use this information to remove
  unnecessary implicit dependencies.

  See also the :prop_tgt:`OPTIMIZE_DEPENDENCIES` target property, which may
  provide another way for reducing the impact of target dependencies in some
  scenarios.

Examples: Generating Files
^^^^^^^^^^^^^^^^^^^^^^^^^^

Custom commands may be used to generate source files.
For example, the code:

.. code-block:: cmake

  add_custom_command(
    OUTPUT out.c
    COMMAND someTool -i ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
                     -o out.c
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
    VERBATIM)
  add_library(myLib out.c)

adds a custom command to run ``someTool`` to generate ``out.c`` and then
compile the generated source as part of a library.  The generation rule
will re-run whenever ``in.txt`` changes.

.. versionadded:: 3.20
  One may use generator expressions to specify per-configuration outputs.
  For example, the code:

  .. code-block:: cmake

    add_custom_command(
      OUTPUT "out-$<CONFIG>.c"
      COMMAND someTool -i ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
                       -o "out-$<CONFIG>.c"
                       -c "$<CONFIG>"
      DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
      VERBATIM)
    add_library(myLib "out-$<CONFIG>.c")

  adds a custom command to run ``someTool`` to generate ``out-<config>.c``,
  where ``<config>`` is the build configuration, and then compile the generated
  source as part of a library.

Example: Generating Files for Multiple Targets
""""""""""""""""""""""""""""""""""""""""""""""

If multiple independent targets need the same custom command output,
it must be attached to a single custom target on which they all depend.
Consider the following example:

.. code-block:: cmake

  add_custom_command(
    OUTPUT table.csv
    COMMAND makeTable -i ${CMAKE_CURRENT_SOURCE_DIR}/input.dat
                      -o table.csv
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/input.dat
    VERBATIM)
  add_custom_target(generate_table_csv DEPENDS table.csv)

  add_custom_command(
    OUTPUT foo.cxx
    COMMAND genFromTable -i table.csv -case foo -o foo.cxx
    DEPENDS table.csv           # file-level dependency
            generate_table_csv  # target-level dependency
    VERBATIM)
  add_library(foo foo.cxx)

  add_custom_command(
    OUTPUT bar.cxx
    COMMAND genFromTable -i table.csv -case bar -o bar.cxx
    DEPENDS table.csv           # file-level dependency
            generate_table_csv  # target-level dependency
    VERBATIM)
  add_library(bar bar.cxx)

Output ``foo.cxx`` is needed only by target ``foo`` and output ``bar.cxx``
is needed only by target ``bar``, but *both* targets need ``table.csv``,
transitively.  Since ``foo`` and ``bar`` are independent targets that may
build concurrently, we prevent them from racing to generate ``table.csv``
by placing its custom command in a separate target, ``generate_table_csv``.
The custom commands generating ``foo.cxx`` and ``bar.cxx`` each specify a
target-level dependency on ``generate_table_csv``, so the targets using them,
``foo`` and ``bar``, will not build until after target ``generate_table_csv``
is built.

.. _`add_custom_command(TARGET)`:

イベントのビルド
^^^^^^^^^^^^^^^^

The second signature adds a custom command to a target such as a
library or executable.  This is useful for performing an operation
before or after building the target.  The command becomes part of the
target and will only execute when the target itself is built.  If the
target is already built, the command will not execute.

.. code-block:: cmake

  add_custom_command(TARGET <target>
                     PRE_BUILD | PRE_LINK | POST_BUILD
                     COMMAND command1 [ARGS] [args1...]
                     [COMMAND command2 [ARGS] [args2...] ...]
                     [BYPRODUCTS [files...]]
                     [WORKING_DIRECTORY dir]
                     [COMMENT comment]
                     [VERBATIM]
                     [COMMAND_EXPAND_LISTS])

This defines a new command that will be associated with building the
specified ``<target>``.  The ``<target>`` must be defined in the current
directory; targets defined in other directories may not be specified.

When the command will happen is determined by which
of the following is specified:

``PRE_BUILD``
  This option has unique behavior for the :ref:`Visual Studio Generators`.
  When using one of the Visual Studio generators, the command will run before
  any other rules are executed within the target.  With all other generators,
  this option behaves the same as ``PRE_LINK`` instead.  Because of this,
  it is recommended to avoid using ``PRE_BUILD`` except when it is known that
  a Visual Studio generator is being used.
``PRE_LINK``
  Run after sources have been compiled but before linking the binary
  or running the librarian or archiver tool of a static library.
  This is not defined for targets created by the
  :command:`add_custom_target` command.
``POST_BUILD``
  Run after all other rules within the target have been executed.

Projects should always specify one of the above three keywords when using
the ``TARGET`` form.  For backward compatibility reasons, ``POST_BUILD`` is
assumed if no such keyword is given, but projects should explicitly provide
one of the keywords to make clear the behavior they expect.

.. note::
  Because generator expressions can be used in custom commands,
  it is possible to define ``COMMAND`` lines or whole custom commands
  which evaluate to empty strings for certain configurations.
  For **Visual Studio 12 2013 (and newer)** generators these command
  lines or custom commands will be omitted for the specific
  configuration and no "empty-string-command" will be added.

  This allows to add individual build events for every configuration.

.. versionadded:: 3.21
  Support for target-dependent generator expressions.

Examples: Build Events
^^^^^^^^^^^^^^^^^^^^^^

A ``POST_BUILD`` event may be used to post-process a binary after linking.
For example, the code:

.. code-block:: cmake

  add_executable(myExe myExe.c)
  add_custom_command(
    TARGET myExe POST_BUILD
    COMMAND someHasher -i "$<TARGET_FILE:myExe>"
                       -o "$<TARGET_FILE:myExe>.hash"
    VERBATIM)

will run ``someHasher`` to produce a ``.hash`` file next to the executable
after linking.

.. versionadded:: 3.20
  One may use generator expressions to specify per-configuration byproducts.
  For example, the code:

  .. code-block:: cmake

    add_library(myPlugin MODULE myPlugin.c)
    add_custom_command(
      TARGET myPlugin POST_BUILD
      COMMAND someHasher -i "$<TARGET_FILE:myPlugin>"
                         --as-code "myPlugin-hash-$<CONFIG>.c"
      BYPRODUCTS "myPlugin-hash-$<CONFIG>.c"
      VERBATIM)
    add_executable(myExe myExe.c "myPlugin-hash-$<CONFIG>.c")

  will run ``someHasher`` after linking ``myPlugin``, e.g. to produce a ``.c``
  file containing code to check the hash of ``myPlugin`` that the ``myExe``
  executable can use to verify it before loading.

Ninja Multi-Config
^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.20

  ``add_custom_command`` supports the :generator:`Ninja Multi-Config`
  generator's cross-config capabilities. See the generator documentation
  for more information.

参考情報
^^^^^^^^

* :command:`add_custom_target`
