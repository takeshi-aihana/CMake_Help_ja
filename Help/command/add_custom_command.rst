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
詳細は「`例：複数のターゲット毎にファイルを生成する`_」のサンプルを参照して下さい。

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

  *この機能の背景については* :policy:`CMP0058` *のポリシーを参照して下さい* 。

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

  ジェネレータ式から得られた次のターゲットのいずれかをコマンドとして指定するか、またはコマンドの引数として指定すると、自動的に「ターゲット・レベル」の依存関係を追加し、コマンドラインを実行する前に、追加した依存先の実行形式を先にビルドする（:policy:`CMP0112` のポリシーも参照のこと）：

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

    この ``IMPLICIT_DEPENDS`` オプションは ``DEPFILE`` オプションと同時には指定することはできない。

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

  ``COMMAND`` のコマンドラインは、可能であれば、端末（``console``）に直接アクセスできる。
  これにより :generator:`Ninja` ジェネレータは、コマンドを ``console`` の  :prop_gbl:`JOB_POOLS` に配置できる。

``VERBATIM``
  ``COMMAND`` のコマンドラインに対するすべての引数 ``args1 args2 ...`` がビルド・ツールのために適切にエスケープされるので、呼び出されるコマンドラインは加工されていない「素」の引数を受け取れる。
  ただし、この ``add_custom_command`` コマンドが引数を受け取るよりも前に :manual:`CMake language <cmake-language(7)>` のプリプロセッサによって一段目のエスケープが解釈されている点に注意すること。
  正しく解釈するためには、この ``VERBATIM`` オプションの使用が推奨されている。
  この ``VERBATIM`` オプションを指定しない場合、引数を解釈する結果は CMake を実行するプラットフォームに依存する。

``WORKING_DIRECTORY``
  ``COMMAND`` のコマンドラインを ``dir`` のディレクトリで実行する。
  ``dir`` に相対パスを指定すると、:variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリとした絶対パスとして解釈される。

  .. versionadded:: 3.13
    ``WORKING_DIRECTORY`` オプションに渡す引数に :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を指定できるようになった。

``DEPFILE``
  .. versionadded:: 3.7

  ``COMMAND`` の依存関係を保持する ``depfile`` を指定する。
  通常は ``COMMAND`` のコマンドラインによって提供されるファイルである。
  このオプションは、ビルドシステムのジェネレータが ``depfile`` をサポートしている場合にのみ使用できる。

  ここで CMake が期待する ``depfile`` のフォーマットは ``gcc`` で ``-M`` オプションを指定して出力されるものと互換があり、使用するジェネレータやプラットフォームに依存したものではない。

  `BNF <https://en.wikipedia.org/wiki/Backus%E2%80%93Naur_form>`_ 記法を使った正式の構文は次のとおり：

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

     ``pathname`` の中にあるスラッシュ（``/``）やバックスラッシュ（``\``）はディレクトリの区切り文字として解釈される。

  .. versionadded:: 3.7
    :generator:`Ninja` ジェネレータは全てのバージョンで、この ``DEPFILE`` オプションをサポートしている。

  .. versionadded:: 3.17
    この ``DEPFILE`` オプションのサポートを含む :generator:`Ninja Multi-Config` ジェネレータが追加された。

  .. versionadded:: 3.20
    :ref:`Makefile Generators` を追加した。

    .. note::

      :ref:`Makefile Generators` 使用時は、この ``DEPFiLE`` オプションと ``IMPLICIT_DEPENDS`` オプションを同時に指定することはできない。

  .. versionadded:: 3.21
    Visual Studio 2012 以降の :ref:`Visual Studio Generators` ジェネレータと :generator:`Xcode` ジェネレータのサポートが追加された。
    さらに「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」のサポートも追加された。

  上記以外のジェネレータで、 この ``DEPFILE`` オプションを使うとエラーになる。

  ``depfile`` を相対パスで指定すると :variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリとして解釈され、さらに ``depfile`` の中の ``pathname`` などに記述した相対パスも :variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリとしてパスを計算する。
  :policy:`CMP0116` のポリシーも参照のこと（このオプションは :ref:`Makefile Generators` や :ref:`Visual Studio Generators`、そして :generator:`Xcode` ジェネレータに対して常に ``NEW`` の機能として扱われる）。

``DEPENDS_EXPLICIT_ONLY``

  .. versionadded:: 3.27

  ``DEPENDS`` オプションで指定したものが ``COMMAND`` のコマンドラインに必要なすべての依存関係（ファイル）であり、他に暗黙的な依存関係は必要ないことを CMake に伝える。

  このオプションを指定しない場合、ターゲットが ``COMMAND`` に指定したコマンドラインの出力を必要とする場合、CMake はこのコマンドラインがターゲットにとって暗黙的な依存関係に含まれるものとみなす。

  CMake 変数の :variable:`CMAKE_ADD_CUSTOM_COMMAND_DEPENDS_EXPLICIT_ONLY` を ``ON`` にすると、全ての ``COMMAND`` のコマンドラインでこのオプションが有効になる。

  現在 :ref:`Ninja Generators` だげが、このオプションを利用して不要な暗黙的な依存関係を削除している。

  :prop_tgt:`OPTIMIZE_DEPENDENCIES` というターゲットのプロパティも参照のこと（これは一部のシナリオで、ターゲットの依存関係の影響を軽減する別の手段を提供する場合がある）。

例：ファイルを生成する
^^^^^^^^^^^^^^^^^^^^^^

``COMMAND`` に指定したコマンドラインを使用して、ターゲットのソース・ファイルを生成できます。
例えば、次のコードは：

.. code-block:: cmake

  add_custom_command(
    OUTPUT out.c
    COMMAND someTool -i ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
                     -o out.c
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
    VERBATIM)
  add_library(myLib out.c)

``someTool`` を実行し ``out.c`` を生成するコマンドラインを ``add_custom_command`` で追加し、ライブラリの一部としてそのファイルをコンパイルしています。
このコマンドラインは ``in.txt`` ファイルが変更されるたびに実行されます。

.. versionadded:: 3.20
  「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使って、ビルド構成ごとに出力を指定できるようになった。
  例えば、次のコードは：

  .. code-block:: cmake

    add_custom_command(
      OUTPUT "out-$<CONFIG>.c"
      COMMAND someTool -i ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
                       -o "out-$<CONFIG>.c"
                       -c "$<CONFIG>"
      DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
      VERBATIM)
    add_library(myLib "out-$<CONFIG>.c")

  ``someTool`` を実行して ``out-<config>.c`` （``<config>`` は ``Release`` や ``Debug`` などのビルド構成の種類）を生成するコマンドラインを ``add_custom_command`` で追加し、ライブラリの一部としてそのファイルをコンパイルしている。

例：複数のターゲット毎にファイルを生成する
""""""""""""""""""""""""""""""""""""""""""

``COMMAND`` に指定したコマンドラインの出力を、独立した複数のターゲットが必要とする場合、それらのターゲットが依存する単一のターゲットを用意して、それに接続する必要があります。
次のような例を考えてみましょう：

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

二つ目の ``add_custom_command`` で追加したコマンドラインが生成する ``foo.cxx`` はターゲットの ``foo`` でのみ必要で、三つ目の ``add_custom_command`` で追加したコマンドラインが生成する ``bar.cxx`` はターゲットの ``bar`` でのみ必要ですが、 *両方* のターゲットは一つ目の
``add_custom_command`` で追加したコマンドラインが生成する ``table.csv`` が一時的に必要になります。
``foo`` と ``bar`` は同時にビルドできる独立したターゲットであるため、``add_custom_command`` を別のターゲット（たとえば ``generate_table_csv``）向けに呼び出すことで、二つ目と三つ目の ``add_custom_command`` が同じ ``table.csv`` を生成するために「競合状態になる」ことを回避できます。
その場合、``foo.cxx`` と ``bar.cxx`` を生成する ``add_custom_command`` は、それぞれ ``generate_table_csv`` を「ターゲット・レベル」の依存関係に指定することになるので、``generate_table_csv`` というターゲットがビルドされるまで、``foo`` と ``bar`` の二つターゲットはビルドされることはありません。

.. _`add_custom_command(TARGET)`:

いろいろなイベントを追加する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

二つ目は、ライブラリや実行形式といったターゲットに「独自（*Custom*）」コマンドを定義します。
これはターゲットをビルドする前後で何か処理する「イベント」を追加したい場合に便利です。
ここで定義した独自コマンドはターゲットの一部となり、ターゲットをビルドする時にだけ実行されます。
ターゲットが既に存在している場合、この独自コマンドは実行されません。

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

この ``add_custom_command`` は、``<target>`` のビルドに関連付けられる新しい独自コマンドを定義します。
``<target>`` には現在のディレクトリの ``CMakeLists.txt`` でビルドされるターゲットを指定して下さい（他のディレクトリにあるターゲットは指定できません）。

定義した独自コマンドがいつ実行されるかは、次のサブオプションのどれが指定されているかによって決まります：

``PRE_BUILD``
  このサブオプションには :ref:`Visual Studio Generators` 専用の動作が含まれている。
  Visual Studio のジェネレータを使用すると、定義した独自コマンドはターゲット内にある他のルールよりも前に実行される。
  それ以外のジェネレータの場合は ``PRE_LINK`` サブオプションと同じ動作になる。
  このため、Visual Studio のジェネレータを使用することが分かっている場合を除き、このサブオプションの使用を避けることを推奨する。
``PRE_LINK``
  ソース・ファイルがコンパイルされた後、またはバイナリにリンクする前、あるいは静的ライブラリをアーカイブする前に、定義した独自コマンドを実行する。
  これは、 :command:`add_custom_target` で定義したターゲットでは指定できない。
``POST_BUILD``
  ターゲット内にある他のルールが全て実行された後に、定義した独自コマンドを実行する。

``TARGET`` オプションを指定する際は、常に上記のサブオプションのいずれかを指定して下さい。
上記のサブオプションを指定しない場合、下位互換性の理由から ``POST_BUILD`` が指定されたものとみなしますが、期待する動作を明確にするために上記のサブオプションを明示的に指定するようにして下さい。

.. note::
  独自コマンドでは「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を利用できるので、特定のビルド構成では「意図的」に空の文字列になる ``COMMAND`` オプションのコマンドラインや独自コマンドが定義できる。
  一方、 **Visual Studio 12 2013 以上** のジェネレータの場合、特定のビルド構成では ``COMMAND`` のコマンドラインや独自コマンドが省略されてしまうので、このような意図的な空の文字列は追加されない。

  これにより、ビルド構成ごとに独自のコマンドを追加できる。

.. versionadded:: 3.21
  ターゲット依存のジェネレータ式をサポートするようになった。

例：リンク後にイベントを追加する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``POST_BUILD`` サブオプションは、リンクした後の実行形式の後処理で利用できます。
たとえば、次のコードは：

.. code-block:: cmake

  add_executable(myExe myExe.c)
  add_custom_command(
    TARGET myExe POST_BUILD
    COMMAND someHasher -i "$<TARGET_FILE:myExe>"
                       -o "$<TARGET_FILE:myExe>.hash"
    VERBATIM)

リンク後に ``someHasher`` を実行して ``.hash`` ファイルを生成する「イベント」を ``add_custom_command`` で追加しています。

.. versionadded:: 3.20
  「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使用して、ビルド構成ごとの出力を指定できる。
  たとえば、次のコードは：

  .. code-block:: cmake

    add_library(myPlugin MODULE myPlugin.c)
    add_custom_command(
      TARGET myPlugin POST_BUILD
      COMMAND someHasher -i "$<TARGET_FILE:myPlugin>"
                         --as-code "myPlugin-hash-$<CONFIG>.c"
      BYPRODUCTS "myPlugin-hash-$<CONFIG>.c"
      VERBATIM)
    add_executable(myExe myExe.c "myPlugin-hash-$<CONFIG>.c")

  ``myPlugin`` をリンクした後に ``someHaher`` を実行して、``myPlugin`` のハッシュ値をチェックするコードが記載された ``.c`` ファイルを出力する。
  これにより、そのあとにビルドする実行形式 ``myExe`` はハッシュ値を検証できる。

Ninja Multi-Config
^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.20

  ``add_custom_command`` コマンドが :generator:`Ninja Multi-Config` ジェネレータの cross-config の機能をサポートするにようなった。
  詳細は、このジェネレータのドキュメントを参照のこと。

参考情報
^^^^^^^^

* :command:`add_custom_target`
