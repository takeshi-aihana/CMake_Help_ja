add_test
--------

:manual:`ctest(1)` で実行するテストを追加する

.. code-block:: cmake

  add_test(NAME <name> COMMAND <command> [<arg>...]
           [CONFIGURATIONS <config>...]
           [WORKING_DIRECTORY <dir>]
           [COMMAND_EXPAND_LISTS])

``<name>`` というテストを一つ追加します。
``<name>`` には、必要に応じて :ref:`Quoted Argument` または :ref:`Bracket Argument` で表現した任意の文字列を含めることができます。
:policy:`CMP0110` のポリシーも参照して下さい。

CMake は、:command:`enable_testing` コマンドが呼び出された場合にだけテストを生成します。
ただし :module:`CTest` モジュールは自動的に ``enable_testing`` を呼び出します（``BUILD_TESTING`` が ``OFF`` の場合は除く）。

``add_test(NAME)`` で追加したテストは、 :command:`set_property(TEST)` または :command:`set_tests_properties` コマンドで設定されたテスト・プロパティの「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」をサポートしています。
テスト・プロパティは、テストを追加したディレクトリにのみ設定できます。

このコマンドのオプションは次のとおりです：

``COMMAND``
  テストを起動するコマンドラインを指定する。
  引数の ``<command>`` の中に、:command:`add_executable` コマンドで追加したビルド・ターゲットを指定すると、ビルドした実行形式のパスに自動的に置き換えられる。

  ``<command>`` には「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使うことができる。

``CONFIGURATIONS``
  テストの実行を ``<config>`` で指定したビルド構成に制限する。

``WORKING_DIRECTORY``
  テスト・プロパティ :prop_test:`WORKING_DIRECTORY` に、テストを実行するディレクトリを表す ``<dir>`` をセットする。
  このオプションを指定しない場合、追加したテストは :variable:`CMAKE_CURRENT_BINARY_DIR` で実行する。
  ``<dir>`` には「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使うことができる。

``COMMAND_EXPAND_LISTS``
  .. versionadded:: 3.16

  ``COMMAND`` オプションに渡した ``<command>`` の中にある :ref:`リスト <CMake Language Lists>` は、「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を含め、すべて展開する。

ここで追加したテストがコード ``0`` で終了した場合、テストは成功です。
それ以外のコードは、テストが「失敗」したことを表します。
テスト・プロパティの :prop_test:`WILL_FAIL` は、この判定方法を反転します。
ただし、セグメンテーション違反やヒープ・エラーといったシステム規模のエラーが原因でテストが失敗した場合、このテスト・プロパティ ``WILL_FALL`` の値にかかわらず、エラー扱いになる点に注意して下さい。
stdout や stderr に書き込まれた出力は :manual:`ctest(1)` が捕捉します。
:prop_test:`PASS_REGULAR_EXPRESSION` や :prop_test:`FAIL_REGULAR_EXPRESSION` やr :prop_test:`SKIP_REGULAR_EXPRESSION` といったテスト・プロパティはテスト結果（成功または失敗）にのみ作用します。

.. versionadded:: 3.16
  :prop_test:`SKIP_REGULAR_EXPRESSION` というテスト・プロパティを追加した。

このコマンドの使用例：

.. code-block:: cmake

  add_test(NAME mytest
           COMMAND testDriver --config $<CONFIG>
                              --exe $<TARGET_FILE:myexe>)

これは ``mytest`` というテストを作成します。
このテストは、 ``testDriver`` というツールに、ビルド構成と ``myexe`` というビルド・ターゲットで提供される実行形式の絶対パス名を引数として渡して起動されます。

---------------------------------------------------------------------

次のコマンドは、上の使用例よりも柔軟性にかける古い呼び出し方です（非推奨）：

.. code-block:: cmake

  add_test(<name> <command> [<arg>...])

``<command>`` をテストを起動するコマンドラインとする ``<name>`` というテストを追加します。

このコマンドに渡す引数の ``<command>`` は、ビルド・ターゲットの参照はサポートしていません。
さらに、この ``<command>`` は「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」をサポートしていません。
