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

``source1 source2 ...`` のソース・ファイルからビルドする ``<name>`` という実行可能なターゲットを追加します。
``<name>`` はターゲットの論理的な名前で、プロジェクト内で重複しないグローバルな名前にして下さい。
ターゲットとしてビルドされる実際のファイル名は、実行するプラットフォームに基づいて付与されます（たとえば ``<name>.exe`` とか ``<name>`` とか）。

.. versionadded:: 3.1
  このコマンドに渡す ``source1 source2 ...`` に ``$<...>`` のような :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を指定できるようになった。
  利用可能な式について詳細は :manual:`cmake-generator-expressions(7)` を参照のこと。

.. versionadded:: 3.11
  あとで :command:`target_sources` コマンドで ``source1 source2 ...`` を追加する場合、これらの引数を省略できるようになった。

デフォルトで、実行形式のファイルは、このコマンドを呼び出したソースツリーに対応したビルドツリーに相当するディレクトリに作成されます。
この場所を変更する方法については :prop_tgt:`RUNTIME_OUTPUT_DIRECTORY` というターゲット・プロパティのドキュメントを参照して下さい。
``<name>`` を最終的にビルドされるファイル名に変更する方法については :prop_tgt:`OUTPUT_NAME` というターゲット・プロパティのドキュメントを参照して下さい。

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
（プロジェクト内の他のターゲットと同様に参照できるようになります）。
``IMPORTED`` な実行形式は :command:`add_custom_command` などのコマンドからの参照に便利なターゲットです。
プロジェクトの外部にある別の実行形式の詳細は  ``IMPORTED_`` で始まるプロパティを使って指定できます。
このうち、最も重要なプロパティは :prop_tgt:`IMPORTED_LOCATION` とビルド構成毎の :prop_tgt:`IMPORTED_LOCATION_<CONFIG>` です（これらは実行形式の場所を表します）。
詳細は ``IMPORTED_*`` なプロパティのドキュメントを参照して下さい。

実行形式に別名を付ける
^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  add_executable(<name> ALIAS <target>)

Creates an :ref:`Alias Target <Alias Targets>`, such that ``<name>`` can be used to refer to ``<target>`` in subsequent commands.
The ``<name>`` does not appear in the generated buildsystem as a make target.
The ``<target>`` may not be an ``ALIAS``.

.. versionadded:: 3.11
  An ``ALIAS`` can target a ``GLOBAL`` :ref:`Imported Target <Imported Targets>`

.. versionadded:: 3.18
  An ``ALIAS`` can target a non-``GLOBAL`` Imported Target.
  Such alias is scoped to the directory in which it is created and subdirectories.
  The :prop_tgt:`ALIAS_GLOBAL` target property can be used to check if the alias is global or not.

``ALIAS`` targets can be used as targets to read properties from, executables for custom commands and custom targets.
They can also be tested for existence with the regular :command:`if(TARGET)` subcommand.
The ``<name>`` may not be used to modify properties of ``<target>``, that is, it may not be used as the operand of :command:`set_property`, :command:`set_target_properties`, :command:`target_link_libraries` etc.
An ``ALIAS`` target may not be installed or exported.

参考情報
^^^^^^^^

* :command:`add_library`
