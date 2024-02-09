add_custom_target
-----------------

ファイルを出力しないターゲットを追加して、常にビルドされるようにする。

.. code-block:: cmake

  add_custom_target(Name [ALL] [command1 [args1...]]
                    [COMMAND command2 [args2...] ...]
                    [DEPENDS depend depend depend ... ]
                    [BYPRODUCTS [files...]]
                    [WORKING_DIRECTORY dir]
                    [COMMENT comment]
                    [JOB_POOL job_pool]
                    [JOB_SERVER_AWARE <bool>]
                    [VERBATIM] [USES_TERMINAL]
                    [COMMAND_EXPAND_LISTS]
                    [SOURCES src1 [src2...]])

``command1`` や ``command2`` などの独自コマンドを実行する ``Name`` というターゲットを一つ追加します。
このターゲットはファイルを生成しないので、独自コマンドでターゲットの ``Name`` と同じファイルを生成しても、 *そのタイムスタンプは常に無視されます* 。
そのため必要であれば、別の :command:`add_custom_command` コマンドを使用して、このターゲットと依存関係にあるファイルを生成して下さい。
何にも依存するものがないターゲットがデフォルトです。
そのため必要であれば、別の :command:`add_dependencies` コマンドを使用して、このターゲットを他のターゲットの依存関係に追加したり、他のターゲットからの依存関係をこのターゲットに追加して下さい。

指定できるオプションは次のとおりです：

``ALL``
  このターゲットが常にビルドされるようにするために、デフォルトのビルド・ターゲットに追加するよう CMake に伝える  [#add_custom_command_not_all]_。

.. rubric:: Footnotes

.. [#add_custom_command_not_all] 一方 :command:`add_custom_command` で追加したコマンドラインは ``ALL`` では呼び出せません。

``BYPRODUCTS``
  .. versionadded:: 3.2

  この ``add_custom_target`` コマンドによって出力させるファイル ``files ...`` を指定する。
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
  ターゲットのビルド時に実行するコマンドライン（``command2``）を指定する。
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

  コマンドライン（``command2``）とその引数（``args2``）はオプションであり、それらを指定しない場合は空の名前を持つターゲットが追加される。

``COMMENT``
  ビルド時にコマンドラインを実行する前に、指定したメッセージを出力する。

  .. versionadded:: 3.26
    ``COMMENT`` オプションに渡す引数に :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を指定できるようになった。

``DEPENDS``
  この ``add_custom_target`` コマンドを呼び出している  ``CMakeLists.txt`` の中で、:command:`add_custom_command` コマンドを使って追加したコマンドライン
  Reference files and outputs of custom commands created with :command:`add_custom_command` command calls in the same directory (``CMakeLists.txt`` file).
  They will be brought up to date when the target is built.

  .. versionchanged:: 3.16
    A target-level dependency is added if any dependency is a byproduct of a target or any of its build events in the same directory to ensure the byproducts will be available before this target is built.

  Use the :command:`add_dependencies` command to add dependencies on other targets.

``COMMAND_EXPAND_LISTS``
  .. versionadded:: 3.8

  Lists in ``COMMAND`` arguments will be expanded, including those
  created with
  :manual:`generator expressions <cmake-generator-expressions(7)>`,
  allowing ``COMMAND`` arguments such as
  ``${CC} "-I$<JOIN:$<TARGET_PROPERTY:foo,INCLUDE_DIRECTORIES>,;-I>" foo.cc``
  to be properly expanded.

``JOB_POOL``
  .. versionadded:: 3.15

  Specify a :prop_gbl:`pool <JOB_POOLS>` for the :generator:`Ninja`
  generator. Incompatible with ``USES_TERMINAL``, which implies
  the ``console`` pool.
  Using a pool that is not defined by :prop_gbl:`JOB_POOLS` causes
  an error by ninja at build time.

``JOB_SERVER_AWARE``
  .. versionadded:: 3.28

  Specify that the command is GNU Make job server aware.

  For the :generator:`Unix Makefiles`, :generator:`MSYS Makefiles`, and
  :generator:`MinGW Makefiles` generators this will add the ``+`` prefix to the
  recipe line. See the `GNU Make Documentation`_ for more information.

  This option is silently ignored by other generators.

.. _`GNU Make Documentation`: https://www.gnu.org/software/make/manual/html_node/MAKE-Variable.html

``SOURCES``
  Specify additional source files to be included in the custom target.
  Specified source files will be added to IDE project files for
  convenience in editing even if they have no build rules.

``VERBATIM``
  All arguments to the commands will be escaped properly for the
  build tool so that the invoked command receives each argument
  unchanged.  Note that one level of escapes is still used by the
  CMake language processor before ``add_custom_target`` even sees
  the arguments.  Use of ``VERBATIM`` is recommended as it enables
  correct behavior.  When ``VERBATIM`` is not given the behavior
  is platform specific because there is no protection of
  tool-specific special characters.

``USES_TERMINAL``
  .. versionadded:: 3.2

  The command will be given direct access to the terminal if possible.
  With the :generator:`Ninja` generator, this places the command in
  the ``console`` :prop_gbl:`pool <JOB_POOLS>`.

``WORKING_DIRECTORY``
  Execute the command with the given current working directory.
  If it is a relative path it will be interpreted relative to the
  build tree directory corresponding to the current source directory.

  .. versionadded:: 3.13
    Arguments to ``WORKING_DIRECTORY`` may use
    :manual:`generator expressions <cmake-generator-expressions(7)>`.

Ninja Multi-Config
^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.20

  ``add_custom_target`` コマンドが :generator:`Ninja Multi-Config` ジェネレータの cross-config の機能をサポートするにようなった。
  詳細は、このジェネレータのドキュメントを参照のこと。

参考情報
^^^^^^^^

* :command:`add_custom_command`
