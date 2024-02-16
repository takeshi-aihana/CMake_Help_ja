add_executable
--------------

.. only:: html

  .. contents::

ソース・ファイルを指定して、実行可能なファイルをプロジェクトに追加する。

通常の実行形式
^^^^^^^^^^^^^^

.. code-block:: cmake

  add_executable(<name> [WIN32] [MACOSX_BUNDLE]
                 [EXCLUDE_FROM_ALL]
                 [source1] [source2 ...])

ソース・ファイル ``source1 source2 ...`` の :ref:`リスト <CMake Language Lists>` からビルドする ``<name>`` という実行可能なターゲットを追加します。
``<name>`` はターゲットの論理的な名前であり、プロジェクト内で重複しないグローバルな名前にして下さい。
ターゲットとしてビルドされる実際のファイル名は、ランタイムのプラットフォームに基づいて付与されます（たとえば ``<name>.exe`` とか ``<name>`` とか）。

.. versionadded:: 3.1
  このコマンドに渡す ``source1 source2 ...`` に ``$<...>`` のような :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を指定できるようになった。
  利用可能な式について詳細は :manual:`cmake-generator-expressions(7)` を参照のこと。

.. versionadded:: 3.11
  あとで :command:`target_sources` コマンドで ``source1 source2 ...`` を追加する場合、これらの引数を省略できるようになった。

デフォルトで実行形式のファイルは、このコマンドを呼び出したソースツリーに対応したビルドツリーに相当するディレクトリに作成されます。
この場所を変更する方法については :prop_tgt:`RUNTIME_OUTPUT_DIRECTORY` というターゲット・プロパティのドキュメントを参照して下さい。
``<name>`` を、最終的にビルドされるファイル名に変更する方法については :prop_tgt:`OUTPUT_NAME` というターゲット・プロパティのドキュメントを参照して下さい。

``WIN32`` オプションを指定すると、:prop_tgt:`WIN32_EXECUTABLE` というターゲット・プロパティがビルドしたターゲットに付与されます。
ターゲット・プロパティの詳細については「:manual:`スコープごとのプロパティの一覧 <cmake-properties(7)>`」を参照して下さい。

``MACOSX_BUNDLE`` オプションを指定すると、対応するプロパティがビルドしたターゲットに付与されます。
詳細は :prop_tgt:`MACOSX_BUNDLE` というターゲット・プロパティを参照して下さい。

``EXCLUDE_FROM_ALL`` オプションを指定すると、対応するプロパティがビルドしたターゲットに付与されます。
詳細は :prop_tgt:`EXCLUDE_FROM_ALL` というターゲット・プロパティを参照して下さい。

ビルドシステムのプロパティ定義について詳細は :manual:`cmake-buildsystem(7)` を参照して下さい。

ソース・ファイルの一部が前処理されて変更されている時に、IDE から処理する前のソース・ファイルにアクセスできるようにする方法については :prop_sf:`HEADER_FILE_ONLY` というプロパティも参照して下さい。

IMPORTED な実行形式
^^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_executable(<name> IMPORTED [GLOBAL])

:ref:`IMPORTED な実行形式 <Imported Targets>` のターゲットは、プロジェクトの外部にある別の実行形式を参照します。
この場合、このターゲットをビルドするためのルールは生成されず、:prop_tgt:`IMPORTED` というターゲット・プロパティが ``True`` になります。
デフォルトで ``<name>`` のスコープは、これが生成されたディレクトリ以下ですが、``GLOBAL`` オプションを指定するとそのスコープが拡張できます
（すなわちプロジェクト内の他のターゲットと同じように参照できるようになります）。
``IMPORTED`` な実行形式は :command:`add_custom_command` などのコマンドからの参照に便利なターゲットです。
プロジェクトの外部にある別の実行形式の詳細は  ``IMPORTED_`` で始まるプロパティを使って指定できます。
このうち、最も重要なプロパティは :prop_tgt:`IMPORTED_LOCATION` とビルド構成毎の :prop_tgt:`IMPORTED_LOCATION_<CONFIG>` です（これらは実行形式の場所を表します）。
詳細は ``IMPORTED_*`` なプロパティのドキュメントを参照して下さい。

ALIAS な実行形式
^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_executable(<name> ALIAS <target>)

後続のコマンドで ``<name>`` で ``<target>`` を参照できるようにする :ref:`ALIAS な実行形式 <Alias Targets>` のターゲットを作成します。
``<name>`` は、ビルドシステムの中でビルド対象のターゲットとして表示されることはありません。
``<target>`` は ``ALIAS`` なターゲットではない場合があります。

.. versionadded:: 3.11
  ``ALIAS`` なターゲットを ``GLOBAL`` で :ref:`IMPORTED なターゲット <Imported Targets>` にすることができるようになった。

.. versionadded:: 3.18
  ``ALIAS`` なターゲットを ``GLOBAL`` ではない :ref:`IMPORTED なターゲット <Imported Targets>` にすることができるようになった。
  このようなターゲットのスコープは、ターゲットを生成したディレクトリとそのサブディレクトリに限定される。
  :prop_tgt:`ALIAS_GLOBAL` というターゲットのプロパティで、``ALIAS`` なターゲットであるかどうかを確認できる。

``ALIAS`` なターゲットは、各種プロパティを読み取るターゲットとか、:command:`add_custom_command` や :command:`add_custom_target` で指定する ``COMMAND`` として利用できます。
さらに :command:`if(TARGET)` コマンドで、これらのターゲット（実行形式）の存在をテストできます。
ただし ``<name>`` を使って ``<target>`` のプロパティを変更することはできません。つまり、:command:`set_property` や :command:`set_target_properties` や :command:`target_link_libraries` コマンドなどでオペランドには指定できません。
また ``ALIAS`` なターゲットはインスールもエキスポートもできません。

参考情報
^^^^^^^^

* :command:`add_library`
