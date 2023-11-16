.. cmake-manual-description: CMake ビルドシステム・リファレンス

cmake-buildsystem(7)
********************

.. only:: html

   .. contents::

はじめに
========

CMake 系のビルドシステムは論理的な「**ターゲット**」の集まりとして構成されます。
これらのターゲットは、それぞれ一個の実行形式またはライブラリ、あるいは独自のコマンド列を実行するカスタム・ターゲットに相当します。
ターゲット間の依存関係はビルドシステムの中で構築されて、ビルドする順番や修正に応じた再構成のルールを決定します。

バイナリのターゲット
====================

実行形式とライブラリは、それぞれ :command:`add_executable` と :command:`add_library` のコマンドを使って定義されます。
このコマンドの実行結果として得られたバイナリ・ファイルには、ターゲットのプラットフォームに対応した適切な :prop_tgt:`PREFIX` と :prop_tgt:`SUFFIX` と拡張子が付与されます。
バイナリのターゲット間にある依存関係は :command:`target_link_libraries` というコマンドを使用して構築されます：

.. code-block:: cmake

  add_library(archive archive.cpp zip.cpp lzma.cpp)
  add_executable(zipapp zipapp.cpp)
  target_link_libraries(zipapp archive)

この例で、``archive`` は ``STATIC`` ライブラリ（その実体は ``archive.cpp``、``zip.cpp``、そして ``lzma.cpp`` をコンパイルしたオブジェクトを格納したアーカイブ）として定義されています。
そして ``zipapp`` は ``zipapp.cpp`` をコンパイル・リンクすることで生成される実行形式として定義されています。
実行形式の ``zipapp`` をリンクする際は、``archive`` という ``STATIC`` ライブラリをリンクします。

.. _`Binary Executables`:

バイナリの実行形式
------------------

:command:`add_executable` コマンドは一個の実行形式をターゲットとして定義します：

.. code-block:: cmake

  add_executable(mytool mytool.cpp)

ビルド時に任意のコマンド列を実行するルールを生成する :command:`add_custom_command` などは、定数の ``COMMAND`` を「実行形式」、:prop_tgt:`EXECUTABLE <TYPE>` を「ターゲット」として汎用的に利用できます。
この時、生成されるビルドシステムのルールは、任意のコマンド列を実行する前に ``COMMAND`` が指す実行形式を先にビルドすることを保証します。

ライブラリの種類
----------------

.. _`Normal Libraries`:

通常のライブラリ
^^^^^^^^^^^^^^^^

:command:`add_library` コマンドはライブラリの種類を指定しないと、デフォルトの ``STATIC`` ライブラリを定義します。
このライブラリの種類は、次のようにコマンドを実行することで直接指定できます：

.. code-block:: cmake

  add_library(archive SHARED archive.cpp zip.cpp lzma.cpp)

.. code-block:: cmake

  add_library(archive STATIC archive.cpp zip.cpp lzma.cpp)

あるいは :variable:`BUILD_SHARED_LIBS` という変数を有効にすると :command:`add_library` コマンドの挙動を変更して、デフォルトで ``SHARED`` ライブラリをビルドさせることができます。

総じて、ビルドシステムが定義するコンテキストの中では、ライブラリが ``SHARED`` であるか ``STATIC`` であるかはほとんど関係ありません（CMake のコマンド、依存関係の仕様、そしてその他の CMake の API はライブラリの種類を問わず「平等」に機能します）。
``MODULE`` という種類のライブラリは一般的に実行形式とリンクされないという点で他の種類のライブラリとは異なります。すなわち :command:`target_link_libraries` コマンドの引数には入りません。
これはランタイム技術を使用して、プラグインとして読み込まれるライブラリです。
ライブラリがアンマネージドなシンボル（たとえば Windows のリソース DLL、C++/CLI の DLL）をエキスポートしていない場合、CMake は ``SHARED`` ライブラリが一個以上のシンボルをエキスポートしていることを期待するので、ライブラリが ``SHARED`` でないことが要求されます。

.. code-block:: cmake

  add_library(archive MODULE 7z.cpp)

.. _`Apple Frameworks`:

Apple のフレームワーク
""""""""""""""""""""""

MacOS や iOS のフレームワーク [#hint_for_framework_and_bundle_of_ios]_ とそのバンドル [#hint_for_framework_and_bundle_of_ios]_ を生成する際、``SHARED`` ライブラリに :prop_tgt:`FRAMEWORK` というターゲット・プロパティが有効（ ``TRUE`` ）になっている場合があります。
一般的に、この ``FRAMEWORK`` というターゲット・プロパティが有効になったライブラリは、さらに :prop_tgt:`FRAMEWORK_VERSION` というターゲット・プロパティも有効になっているはずです。
このプロパティは通常、MacOS の慣例に倣って "A" の値がセットされます。
``MACOSX_FRAMEWORK_IDENTIFIER`` には  ``CFBundleIdentifier`` キーの値をセットして、バンドルを一意に識別します。

.. code-block:: cmake

  add_library(MyFramework SHARED MyFramework.cpp)
  set_target_properties(MyFramework PROPERTIES
    FRAMEWORK TRUE
    FRAMEWORK_VERSION A # Version "A" is macOS convention
    MACOSX_FRAMEWORK_IDENTIFIER org.cmake.MyFramework
  )

.. _`Object Libraries`:

オブジェクト・ライブラリ
^^^^^^^^^^^^^^^^^^^^^^^^

``OBJECT`` ライブラリという種類は、指定したソース・ファイルをコンパイルして生成したブジェクト・ファイルをアーカイブ化せずに集めたものです。
これらのオブジェクト・ファイルは :genex:`$<TARGET_OBJECTS:name>` という文法で、別のターゲットの入力ソースとして利用することができます。

これは ``OBJECT`` ライブラリの中身（オブジェクト・ファイル）を別のターゲットに提供する際に使用できる :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` の一つです：

.. code-block:: cmake

  add_library(archive OBJECT archive.cpp zip.cpp lzma.cpp)

  add_library(archiveExtras STATIC $<TARGET_OBJECTS:archive> extras.cpp)

  add_executable(test_exe $<TARGET_OBJECTS:archive> test.cpp)

この例では、別のターゲットをビルドする際にそのソースファイルに加え ``OBJECT`` ライブラリのオブジェクト・ファイルを利用しています。

それ以外には、``OBJECT`` ライブラリが別のターゲットとリンクされる場合があります：

.. code-block:: cmake

  add_library(archive OBJECT archive.cpp zip.cpp lzma.cpp)

  add_library(archiveExtras STATIC extras.cpp)
  target_link_libraries(archiveExtras PUBLIC archive)

  add_executable(test_exe test.cpp)
  target_link_libraries(test_exe archive)

この例では、別のターゲットをビルドする際に *直接*  リンクされている ``OBJECT`` ライブラリからオブジェクト・ファイルを利用しています。
なお ``OBJECT`` ライブラリの 「:ref:`利用要件 <Target Usage Requirements>`」（*Usage Requirements*） [#hint_for_build_specification]_ は別のターゲットをビルドする際に優先されます。
その上、「利用要件」は別のターゲットに依存するものに推移的（*Transitive*） [#hint_for_transitive]_ に伝搬していきます。

``OBJECT`` ライブラリは :command:`add_custom_command(TARGET)` のような使い方で ``TARGET`` には指定することはできません。
ただしオブジェクト・ファイルのリストは、``$<TARGET_OBJECTS:objlib>`` を使用して :command:`add_custom_command(OUTPUT)` とか :command:`file(GENERATE)` のコマンドで利用することはできます。

ビルドの仕様と利用要件
======================

:command:`target_include_directories` や :command:`target_compile_definitions` や :command:`target_compile_options` といったコマンドは、バイナリのターゲットに対する「ビルドの仕様」（*Build Specification*） [#hint_for_build_specification]_ と「利用要件」（*Usage Requirements*） [#hint_for_build_specification]_ を指定します。
これらのコマンドは、順に :prop_tgt:`INCLUDE_DIRECTORIES` 、:prop_tgt:`COMPILE_DEFINITIONS` 、そして :prop_tgt:`COMPILE_OPTIONS` というターゲット・プロパティおよび / または :prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES`、:prop_tgt:`INTERFACE_COMPILE_DEFINITIONS`、そして :prop_tgt:`INTERFACE_COMPILE_OPTIONS` というターゲット・プロパティをセットします。

各コマンドには ``PRIVATE``、``PUBLIC``、そして ``INTERFACE`` というモードがあります。
``PRIVATE`` モードは ``INTERFACE_`` 系以外のターゲット・プロパティだけセットし、``INTERFACE`` モードは ``INTERFACE_`` 系のターゲット・プロパティだけをセットし、``PUBLIC`` モードはその両方の系のターゲット・プロパティをセットします。
各コマンドは各モードを複数回使用して呼び出すことができます：

.. code-block:: cmake

  target_compile_definitions(archive
    PRIVATE BUILDING_WITH_LZMA
    INTERFACE USING_ARCHIVE_LIB
  )

利用要件は、ダウンストリームで特定のターゲット・プロパティ、たとえば :prop_tgt:`COMPILE_OPTIONS` や :prop_tgt:`COMPILE_DEFINITIONS` などを利便性のみを目的として使用されることを意図したものではないことに注意して下さい。
これらのプロパティの値は単に使うことが推奨されるとか、使うと便利だとかではなく、**使うことが必須** でなければなりません。

再配布用のパッケージを作成する際に利用要件を指定する場合の追加の留意点については :manual:`cmake-packages(7)` のマニュアルにある 「:ref:`Creating Relocatable Packages` 」というセクションを参照して下さい。

ターゲット・プロパティ
----------------------

:prop_tgt:`INCLUDE_DIRECTORIES` や :prop_tgt:`COMPILE_DEFINITIONS` や :prop_tgt:`COMPILE_OPTIONS` といったターゲット・プロパティのエントリは、バイナリのソース・ファイルをコンパイルする時に適切に使用されます。

この中で :prop_tgt:`INCLUDE_DIRECTORIES` にセットされたエントリは、``-I`` や ``-isystem`` という接頭子を付け、セットされたエントリの出現順にコンパイル行に追加されます。

:prop_tgt:`COMPILE_DEFINITIONS` にセットされたエントリは ``-D`` や ``/D`` という接頭子を付け、順不同でコンパイル行に追加されます。
また :prop_tgt:`DEFINE_SYMBOL` というターゲット・プロパティは ``SHARED`` と ``STATIC`` ライブラリをターゲットとした特別な場合のコンパイル定義として追加されます。

:prop_tgt:`COMPILE_OPTIONS` というターゲット・プロパティのエントリは SHELL 用にエスケープされ、セットされたエントリの出現順に追加されていきます。
その他に、コンパイル・オプションには :prop_tgt:`POSITION_INDEPENDENT_CODE` といった特殊な処理もあります。

ターゲット・プロパティの :prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES` や :prop_tgt:`INTERFACE_COMPILE_DEFINITIONS`、そして :prop_tgt:`INTERFACE_COMPILE_OPTIONS` のエントリは「利用要件」（*Usage Requirements* ） [#hint_for_build_specification]_ です
（これらのプロパティには、利用者が正しくコンパイルしターゲットとリンクするために必要なエントリを指定します）。
バイナリのターゲットの場合は :command:`target_link_libraries` コマンドに指定した各ターゲットで、接頭子 ``INTERFACE_`` が付いたプロパティのエントリをそれぞれ使います：

.. code-block:: cmake

  set(srcs archive.cpp zip.cpp)
  if (LZMA_FOUND)
    list(APPEND srcs lzma.cpp)
  endif()
  add_library(archive SHARED ${srcs})
  if (LZMA_FOUND)
    # The archive library sources are compiled with -DBUILDING_WITH_LZMA
    target_compile_definitions(archive PRIVATE BUILDING_WITH_LZMA)
  endif()
  target_compile_definitions(archive INTERFACE USING_ARCHIVE_LIB)

  add_executable(consumer)
  # Link consumer to archive and consume its usage requirements. The consumer
  # executable sources are compiled with -DUSING_ARCHIVE_LIB.
  target_link_libraries(consumer archive)

ソース・ディレクトリとそれに対応するビルド・ディレクトリが :prop_tgt:`INCLUDE_DIRECTORIES` というターゲット・プロパティに追加されるのが一般的な使い方なので、CMake 変数である :variable:`CMAKE_INCLUDE_CURRENT_DIR` を ``TRUE`` にすると、これらのディレクトリがすべてのターゲットの :prop_tgt:`INCLUDE_DIRECTORIES` プロパティに簡単に追加できます。
CMake 変数の :variable:`CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE` を ``TRUE`` にすると、対応するディレクトリを全てのターゲットの :prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES` プロパティに追加できます。
これにより、:command:`target_link_libraries` コマンドを使って（ビルド・ディレクトリがそれぞれ異なる）複数のターゲットを簡単に扱えるようになります。

.. _`Target Usage Requirements`:

伝搬する利用要件
----------------

ターゲットの「利用要件」は依存先に推移的（*Transitive*） [#hint_for_transitive]_ に伝搬していきます。
:command:`target_link_libraries` コマンドには、この伝搬を制御するために ``PRIVATE``、``INTERFACE``、そして ``PUBLIC`` モードがあります。

.. code-block:: cmake

  add_library(archive archive.cpp)
  target_compile_definitions(archive INTERFACE USING_ARCHIVE_LIB)

  add_library(serialization serialization.cpp)
  target_compile_definitions(serialization INTERFACE USING_SERIALIZATION_LIB)

  add_library(archiveExtras extras.cpp)
  target_link_libraries(archiveExtras PUBLIC archive)
  target_link_libraries(archiveExtras PRIVATE serialization)
  # archiveExtras is compiled with -DUSING_ARCHIVE_LIB
  # and -DUSING_SERIALIZATION_LIB

  add_executable(consumer consumer.cpp)
  # consumer is compiled with -DUSING_ARCHIVE_LIB
  target_link_libraries(consumer archiveExtras)

この例では、``archive`` ライブラリは ``archiveExtras`` ライブラリと ``PUBLIC`` な依存関係にあるので、その利用要件は実行形式の ``consumer`` にも伝搬します。
また、``serialization`` ライブラリは ``archiveExtras`` ライブラリと ``PRIVATE`` な依存関係にあるので、その利用要件は実行形式の ``consumer`` には伝搬しません。

一般に、依存関係がライブラリのビルドのみで利用され、ヘッダ・ファイルには影響しないような場合は ``PRIVATE`` モードで :command:`target_link_libraries` コマンドを呼び出す時にその依存関係を指定するようにして下さい。
もし依存関係がライブラリのヘッダ・ファイルの中で追加でインクルードされる場合（たとえばクラスの継承）は ``PUBLIC`` モードで依存関係を指定して下さい。
ライブラリの実装で利用されず、ヘッダ・ファイルのみ利用される依存関係の場合は ``INTERFACE`` モードで依存関係を指定して下さい。
:command:`target_link_libraries` コマンドは各モードを複数回指定して呼び出すことも可能です：

.. code-block:: cmake

  target_link_libraries(archiveExtras
    PUBLIC archive
    PRIVATE serialization
  )

利用要件は、依存関係から ``INTERFACE_`` 系のターゲット・プロパティを読み取り、そのエントリを依存先の ``INTERFACE_`` 系プロパティの最後に追加することによって伝搬していきます。
たとえば、依存元のプロパティである :prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES` を読み取って、そのエントリを依存先のプロパティの :prop_tgt:`INCLUDE_DIRECTORIES` に追加していきます。

この時、追加した順番が適切なのに :command:`target_link_libraries` コマンドの呼び出し結果だとコンパイルが失敗する場合、妥当なコマンドを使ってプロパティを直接セットして順番を更新できる場合があります。
たとえば、ターゲットにライブラリをリンクする際に ``lib1`` ``lib2`` ``lib3`` の順番でリンクし、:prop_tgt:`INCLUDE_DIRECTORIES` プロパティでは ``lib3`` ``lib1`` ``lib2`` の順番で指定したい場合は、次のようになります：

.. code-block:: cmake

  target_link_libraries(myExe lib1 lib2 lib3)
  target_include_directories(myExe
    PRIVATE $<TARGET_PROPERTY:lib3,INTERFACE_INCLUDE_DIRECTORIES>)

ただし :command:`install(EXPORT)` コマンドでインストールして、外部に公開するターゲットの利用要件を指定する場合は注意が必要です。
詳細は「:ref:`Creating Packages`」を参照して下さい。

.. _`Compatible Interface Properties`:

互換性のあるインタフェースのプロパティ
--------------------------------------

一部のターゲット・プロパティは、ターゲットと依存関係のインタフェースとの間で互換性を持つものがあります。
たとえば :prop_tgt:`POSITION_INDEPENDENT_CODE` というターゲット・プロパティは、ターゲットが PIC（*Position Independent Code* ：位置独立コード）としてコンパイルすべきかどうかを表す論理値を指定します（つまり、このプロパティはプラットフォーム依存です）。
一方、ターゲット側は利用要件のプロパティである :prop_tgt:`INTERFACE_POSITION_INDEPENDENT_CODE` を使って、利用者に PIC としてコンパイルすべきかどうかを伝えることができます。

.. code-block:: cmake

  add_executable(exe1 exe1.cpp)
  set_property(TARGET exe1 PROPERTY POSITION_INDEPENDENT_CODE ON)

  add_library(lib1 SHARED lib1.cpp)
  set_property(TARGET lib1 PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE ON)

  add_executable(exe2 exe2.cpp)
  target_link_libraries(exe2 lib1)

この例では ``exe1`` と ``exe2`` の両方の実行形式が PIC としてコンパイルされます。
一方 ``lib1`` ライブラリも PIC としてコンパイルされます。なぜなら、このライブラリはデフォルトで ``SHARED`` ライブラリだからです。
もし依存関係が競合して互換性がない場合は :manual:`cmake(1)` はエラーを出力します：

.. code-block:: cmake

  add_library(lib1 SHARED lib1.cpp)
  set_property(TARGET lib1 PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE ON)

  add_library(lib2 SHARED lib2.cpp)
  set_property(TARGET lib2 PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE OFF)

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 lib1)
  set_property(TARGET exe1 PROPERTY POSITION_INDEPENDENT_CODE OFF)

  add_executable(exe2 exe2.cpp)
  target_link_libraries(exe2 lib1 lib2)

この例で、``lib1`` ライブラリの利用要件である ``INTERFACE_POSITION_INDEPENDENT_CODE`` プロパティはターゲットである ``exe1`` の :prop_tgt:`POSITION_INDEPENDENT_CODE` プロパティとは「互換性」はありません。
ライブラリは、その利用者が PIC としてビルドすることが期待されますが、その一方で実行形式は PIC としてビルドされないことが期待されるためエラーになります。

The ``lib1`` and ``lib2`` requirements are not "compatible".  One of them requires that consumers are built as position-independent-code, while the other requires that consumers are not built as position-independent-code.
Because ``exe2`` links to both and they are in conflict, a CMake error message is issued::

  CMake Error: The INTERFACE_POSITION_INDEPENDENT_CODE property of "lib2" does
  not agree with the value of POSITION_INDEPENDENT_CODE already determined
  for "exe2".

To be "compatible", the :prop_tgt:`POSITION_INDEPENDENT_CODE` property, if set must be either the same, in a boolean sense, as the :prop_tgt:`INTERFACE_POSITION_INDEPENDENT_CODE` property of all transitively specified dependencies on which that property is set.

This property of "compatible interface requirement" may be extended to other properties by specifying the property in the content of the :prop_tgt:`COMPATIBLE_INTERFACE_BOOL` target property.
Each specified property  must be compatible between the consuming target and the corresponding property with an ``INTERFACE_`` prefix from each dependency:

.. code-block:: cmake

  add_library(lib1Version2 SHARED lib1_v2.cpp)
  set_property(TARGET lib1Version2 PROPERTY INTERFACE_CUSTOM_PROP ON)
  set_property(TARGET lib1Version2 APPEND PROPERTY
    COMPATIBLE_INTERFACE_BOOL CUSTOM_PROP
  )

  add_library(lib1Version3 SHARED lib1_v3.cpp)
  set_property(TARGET lib1Version3 PROPERTY INTERFACE_CUSTOM_PROP OFF)

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 lib1Version2) # CUSTOM_PROP will be ON

  add_executable(exe2 exe2.cpp)
  target_link_libraries(exe2 lib1Version2 lib1Version3) # Diagnostic

Non-boolean properties may also participate in "compatible interface" computations.
Properties specified in the :prop_tgt:`COMPATIBLE_INTERFACE_STRING` property must be either unspecified or compare to the same string among all transitively specified dependencies.
This can be useful to ensure that multiple incompatible versions of a library are not linked together through transitive requirements of a target:

.. code-block:: cmake

  add_library(lib1Version2 SHARED lib1_v2.cpp)
  set_property(TARGET lib1Version2 PROPERTY INTERFACE_LIB_VERSION 2)
  set_property(TARGET lib1Version2 APPEND PROPERTY
    COMPATIBLE_INTERFACE_STRING LIB_VERSION
  )

  add_library(lib1Version3 SHARED lib1_v3.cpp)
  set_property(TARGET lib1Version3 PROPERTY INTERFACE_LIB_VERSION 3)

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 lib1Version2) # LIB_VERSION will be "2"

  add_executable(exe2 exe2.cpp)
  target_link_libraries(exe2 lib1Version2 lib1Version3) # Diagnostic

The :prop_tgt:`COMPATIBLE_INTERFACE_NUMBER_MAX` target property specifies that content will be evaluated numerically and the maximum number among all specified will be calculated:

.. code-block:: cmake

  add_library(lib1Version2 SHARED lib1_v2.cpp)
  set_property(TARGET lib1Version2 PROPERTY INTERFACE_CONTAINER_SIZE_REQUIRED 200)
  set_property(TARGET lib1Version2 APPEND PROPERTY
    COMPATIBLE_INTERFACE_NUMBER_MAX CONTAINER_SIZE_REQUIRED
  )

  add_library(lib1Version3 SHARED lib1_v3.cpp)
  set_property(TARGET lib1Version3 PROPERTY INTERFACE_CONTAINER_SIZE_REQUIRED 1000)

  add_executable(exe1 exe1.cpp)
  # CONTAINER_SIZE_REQUIRED will be "200"
  target_link_libraries(exe1 lib1Version2)

  add_executable(exe2 exe2.cpp)
  # CONTAINER_SIZE_REQUIRED will be "1000"
  target_link_libraries(exe2 lib1Version2 lib1Version3)

Similarly, the :prop_tgt:`COMPATIBLE_INTERFACE_NUMBER_MIN` may be used to calculate the numeric minimum value for a property from dependencies.

Each calculated "compatible" property value may be read in the consumer at generate-time using generator expressions.

Note that for each dependee, the set of properties specified in each compatible interface property must not intersect with the set specified in any of the other properties.

Property Origin Debugging
-------------------------

Because build specifications can be determined by dependencies, the lack of
locality of code which creates a target and code which is responsible for
setting build specifications may make the code more difficult to reason about.
:manual:`cmake(1)` provides a debugging facility to print the origin of the
contents of properties which may be determined by dependencies.  The properties
which can be debugged are listed in the
:variable:`CMAKE_DEBUG_TARGET_PROPERTIES` variable documentation:

.. code-block:: cmake

  set(CMAKE_DEBUG_TARGET_PROPERTIES
    INCLUDE_DIRECTORIES
    COMPILE_DEFINITIONS
    POSITION_INDEPENDENT_CODE
    CONTAINER_SIZE_REQUIRED
    LIB_VERSION
  )
  add_executable(exe1 exe1.cpp)

In the case of properties listed in :prop_tgt:`COMPATIBLE_INTERFACE_BOOL` or
:prop_tgt:`COMPATIBLE_INTERFACE_STRING`, the debug output shows which target
was responsible for setting the property, and which other dependencies also
defined the property.  In the case of
:prop_tgt:`COMPATIBLE_INTERFACE_NUMBER_MAX` and
:prop_tgt:`COMPATIBLE_INTERFACE_NUMBER_MIN`, the debug output shows the
value of the property from each dependency, and whether the value determines
the new extreme.

Build Specification with Generator Expressions
----------------------------------------------

Build specifications may use
:manual:`generator expressions <cmake-generator-expressions(7)>` containing
content which may be conditional or known only at generate-time.  For example,
the calculated "compatible" value of a property may be read with the
``TARGET_PROPERTY`` expression:

.. code-block:: cmake

  add_library(lib1Version2 SHARED lib1_v2.cpp)
  set_property(TARGET lib1Version2 PROPERTY
    INTERFACE_CONTAINER_SIZE_REQUIRED 200)
  set_property(TARGET lib1Version2 APPEND PROPERTY
    COMPATIBLE_INTERFACE_NUMBER_MAX CONTAINER_SIZE_REQUIRED
  )

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 lib1Version2)
  target_compile_definitions(exe1 PRIVATE
      CONTAINER_SIZE=$<TARGET_PROPERTY:CONTAINER_SIZE_REQUIRED>
  )

In this case, the ``exe1`` source files will be compiled with
``-DCONTAINER_SIZE=200``.

The unary ``TARGET_PROPERTY`` generator expression and the ``TARGET_POLICY``
generator expression are evaluated with the consuming target context.  This
means that a usage requirement specification may be evaluated differently based
on the consumer:

.. code-block:: cmake

  add_library(lib1 lib1.cpp)
  target_compile_definitions(lib1 INTERFACE
    $<$<STREQUAL:$<TARGET_PROPERTY:TYPE>,EXECUTABLE>:LIB1_WITH_EXE>
    $<$<STREQUAL:$<TARGET_PROPERTY:TYPE>,SHARED_LIBRARY>:LIB1_WITH_SHARED_LIB>
    $<$<TARGET_POLICY:CMP0041>:CONSUMER_CMP0041_NEW>
  )

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 lib1)

  cmake_policy(SET CMP0041 NEW)

  add_library(shared_lib shared_lib.cpp)
  target_link_libraries(shared_lib lib1)

The ``exe1`` executable will be compiled with ``-DLIB1_WITH_EXE``, while the
``shared_lib`` shared library will be compiled with ``-DLIB1_WITH_SHARED_LIB``
and ``-DCONSUMER_CMP0041_NEW``, because policy :policy:`CMP0041` is
``NEW`` at the point where the ``shared_lib`` target is created.

The ``BUILD_INTERFACE`` expression wraps requirements which are only used when
consumed from a target in the same buildsystem, or when consumed from a target
exported to the build directory using the :command:`export` command.  The
``INSTALL_INTERFACE`` expression wraps requirements which are only used when
consumed from a target which has been installed and exported with the
:command:`install(EXPORT)` command:

.. code-block:: cmake

  add_library(ClimbingStats climbingstats.cpp)
  target_compile_definitions(ClimbingStats INTERFACE
    $<BUILD_INTERFACE:ClimbingStats_FROM_BUILD_LOCATION>
    $<INSTALL_INTERFACE:ClimbingStats_FROM_INSTALLED_LOCATION>
  )
  install(TARGETS ClimbingStats EXPORT libExport ${InstallArgs})
  install(EXPORT libExport NAMESPACE Upstream::
          DESTINATION lib/cmake/ClimbingStats)
  export(EXPORT libExport NAMESPACE Upstream::)

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 ClimbingStats)

In this case, the ``exe1`` executable will be compiled with
``-DClimbingStats_FROM_BUILD_LOCATION``.  The exporting commands generate
:prop_tgt:`IMPORTED` targets with either the ``INSTALL_INTERFACE`` or the
``BUILD_INTERFACE`` omitted, and the ``*_INTERFACE`` marker stripped away.
A separate project consuming the ``ClimbingStats`` package would contain:

.. code-block:: cmake

  find_package(ClimbingStats REQUIRED)

  add_executable(Downstream main.cpp)
  target_link_libraries(Downstream Upstream::ClimbingStats)

Depending on whether the ``ClimbingStats`` package was used from the build
location or the install location, the ``Downstream`` target would be compiled
with either ``-DClimbingStats_FROM_BUILD_LOCATION`` or
``-DClimbingStats_FROM_INSTALL_LOCATION``.  For more about packages and
exporting see the :manual:`cmake-packages(7)` manual.

.. _`Include Directories and Usage Requirements`:

Include Directories and Usage Requirements
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Include directories require some special consideration when specified as usage
requirements and when used with generator expressions.  The
:command:`target_include_directories` command accepts both relative and
absolute include directories:

.. code-block:: cmake

  add_library(lib1 lib1.cpp)
  target_include_directories(lib1 PRIVATE
    /absolute/path
    relative/path
  )

Relative paths are interpreted relative to the source directory where the
command appears.  Relative paths are not allowed in the
:prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES` of :prop_tgt:`IMPORTED` targets.

In cases where a non-trivial generator expression is used, the
``INSTALL_PREFIX`` expression may be used within the argument of an
``INSTALL_INTERFACE`` expression.  It is a replacement marker which
expands to the installation prefix when imported by a consuming project.

Include directories usage requirements commonly differ between the build-tree
and the install-tree.  The ``BUILD_INTERFACE`` and ``INSTALL_INTERFACE``
generator expressions can be used to describe separate usage requirements
based on the usage location.  Relative paths are allowed within the
``INSTALL_INTERFACE`` expression and are interpreted relative to the
installation prefix.  For example:

.. code-block:: cmake

  add_library(ClimbingStats climbingstats.cpp)
  target_include_directories(ClimbingStats INTERFACE
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/generated>
    $<INSTALL_INTERFACE:/absolute/path>
    $<INSTALL_INTERFACE:relative/path>
    $<INSTALL_INTERFACE:$<INSTALL_PREFIX>/$<CONFIG>/generated>
  )

Two convenience APIs are provided relating to include directories usage
requirements.  The :variable:`CMAKE_INCLUDE_CURRENT_DIR_IN_INTERFACE` variable
may be enabled, with an equivalent effect to:

.. code-block:: cmake

  set_property(TARGET tgt APPEND PROPERTY INTERFACE_INCLUDE_DIRECTORIES
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR};${CMAKE_CURRENT_BINARY_DIR}>
  )

for each target affected.  The convenience for installed targets is
an ``INCLUDES DESTINATION`` component with the :command:`install(TARGETS)`
command:

.. code-block:: cmake

  install(TARGETS foo bar bat EXPORT tgts ${dest_args}
    INCLUDES DESTINATION include
  )
  install(EXPORT tgts ${other_args})
  install(FILES ${headers} DESTINATION include)

This is equivalent to appending ``${CMAKE_INSTALL_PREFIX}/include`` to the
:prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES` of each of the installed
:prop_tgt:`IMPORTED` targets when generated by :command:`install(EXPORT)`.

When the :prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES` of an
:ref:`imported target <Imported targets>` is consumed, the entries in the
property may be treated as system include directories.  The effects of that
are toolchain-dependent, but one common effect is to omit compiler warnings
for headers found in those directories.  The :prop_tgt:`SYSTEM` property of
the installed target determines this behavior (see the
:prop_tgt:`EXPORT_NO_SYSTEM` property for how to modify the installed value
for a target).  It is also possible to change how consumers interpret the
system behavior of consumed imported targets by setting the
:prop_tgt:`NO_SYSTEM_FROM_IMPORTED` target property on the *consumer*.

If a binary target is linked transitively to a macOS :prop_tgt:`FRAMEWORK`, the
``Headers`` directory of the framework is also treated as a usage requirement.
This has the same effect as passing the framework directory as an include
directory.

Link Libraries and Generator Expressions
----------------------------------------

Like build specifications, :prop_tgt:`link libraries <LINK_LIBRARIES>` may be
specified with generator expression conditions.  However, as consumption of
usage requirements is based on collection from linked dependencies, there is
an additional limitation that the link dependencies must form a "directed
acyclic graph".  That is, if linking to a target is dependent on the value of
a target property, that target property may not be dependent on the linked
dependencies:

.. code-block:: cmake

  add_library(lib1 lib1.cpp)
  add_library(lib2 lib2.cpp)
  target_link_libraries(lib1 PUBLIC
    $<$<TARGET_PROPERTY:POSITION_INDEPENDENT_CODE>:lib2>
  )
  add_library(lib3 lib3.cpp)
  set_property(TARGET lib3 PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE ON)

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 lib1 lib3)

As the value of the :prop_tgt:`POSITION_INDEPENDENT_CODE` property of
the ``exe1`` target is dependent on the linked libraries (``lib3``), and the
edge of linking ``exe1`` is determined by the same
:prop_tgt:`POSITION_INDEPENDENT_CODE` property, the dependency graph above
contains a cycle.  :manual:`cmake(1)` issues an error message.

.. _`Output Artifacts`:

Output Artifacts
----------------

The buildsystem targets created by the :command:`add_library` and
:command:`add_executable` commands create rules to create binary outputs.
The exact output location of the binaries can only be determined at
generate-time because it can depend on the build-configuration and the
link-language of linked dependencies etc.  ``TARGET_FILE``,
``TARGET_LINKER_FILE`` and related expressions can be used to access the
name and location of generated binaries.  These expressions do not work
for ``OBJECT`` libraries however, as there is no single file generated
by such libraries which is relevant to the expressions.

There are three kinds of output artifacts that may be build by targets
as detailed in the following sections.  Their classification differs
between DLL platforms and non-DLL platforms.  All Windows-based
systems including Cygwin are DLL platforms.

.. _`Runtime Output Artifacts`:

Runtime Output Artifacts
^^^^^^^^^^^^^^^^^^^^^^^^

A *runtime* output artifact of a buildsystem target may be:

* The executable file (e.g. ``.exe``) of an executable target
  created by the :command:`add_executable` command.

* On DLL platforms: the executable file (e.g. ``.dll``) of a shared
  library target created by the :command:`add_library` command
  with the ``SHARED`` option.

The :prop_tgt:`RUNTIME_OUTPUT_DIRECTORY` and :prop_tgt:`RUNTIME_OUTPUT_NAME`
target properties may be used to control runtime output artifact locations
and names in the build tree.

.. _`Library Output Artifacts`:

Library Output Artifacts
^^^^^^^^^^^^^^^^^^^^^^^^

A *library* output artifact of a buildsystem target may be:

* The loadable module file (e.g. ``.dll`` or ``.so``) of a module
  library target created by the :command:`add_library` command
  with the ``MODULE`` option.

* On non-DLL platforms: the shared library file (e.g. ``.so`` or ``.dylib``)
  of a shared library target created by the :command:`add_library`
  command with the ``SHARED`` option.

The :prop_tgt:`LIBRARY_OUTPUT_DIRECTORY` and :prop_tgt:`LIBRARY_OUTPUT_NAME`
target properties may be used to control library output artifact locations
and names in the build tree.

.. _`Archive Output Artifacts`:

Archive Output Artifacts
^^^^^^^^^^^^^^^^^^^^^^^^

An *archive* output artifact of a buildsystem target may be:

* The static library file (e.g. ``.lib`` or ``.a``) of a static
  library target created by the :command:`add_library` command
  with the ``STATIC`` option.

* On DLL platforms: the import library file (e.g. ``.lib``) of a shared
  library target created by the :command:`add_library` command
  with the ``SHARED`` option.  This file is only guaranteed to exist if
  the library exports at least one unmanaged symbol.

* On DLL platforms: the import library file (e.g. ``.lib``) of an
  executable target created by the :command:`add_executable` command
  when its :prop_tgt:`ENABLE_EXPORTS` target property is set.

* On AIX: the linker import file (e.g. ``.imp``) of an executable target
  created by the :command:`add_executable` command when its
  :prop_tgt:`ENABLE_EXPORTS` target property is set.

* On macOS: the linker import file (e.g. ``.tbd``) of a shared library target
  created by the :command:`add_library` command with the ``SHARED`` option and
  when its :prop_tgt:`ENABLE_EXPORTS` target property is set.

The :prop_tgt:`ARCHIVE_OUTPUT_DIRECTORY` and :prop_tgt:`ARCHIVE_OUTPUT_NAME`
target properties may be used to control archive output artifact locations
and names in the build tree.

Directory-Scoped Commands
-------------------------

The :command:`target_include_directories`,
:command:`target_compile_definitions` and
:command:`target_compile_options` commands have an effect on only one
target at a time.  The commands :command:`add_compile_definitions`,
:command:`add_compile_options` and :command:`include_directories` have
a similar function, but operate at directory scope instead of target
scope for convenience.

.. _`Build Configurations`:

Build Configurations
====================

Configurations determine specifications for a certain type of build, such
as ``Release`` or ``Debug``.  The way this is specified depends on the type
of :manual:`generator <cmake-generators(7)>` being used.  For single
configuration generators like  :ref:`Makefile Generators` and
:generator:`Ninja`, the configuration is specified at configure time by the
:variable:`CMAKE_BUILD_TYPE` variable. For multi-configuration generators
like :ref:`Visual Studio <Visual Studio Generators>`, :generator:`Xcode`, and
:generator:`Ninja Multi-Config`, the configuration is chosen by the user at
build time and :variable:`CMAKE_BUILD_TYPE` is ignored.  In the
multi-configuration case, the set of *available* configurations is specified
at configure time by the :variable:`CMAKE_CONFIGURATION_TYPES` variable,
but the actual configuration used cannot be known until the build stage.
This difference is often misunderstood, leading to problematic code like the
following:

.. code-block:: cmake

  # WARNING: This is wrong for multi-config generators because they don't use
  #          and typically don't even set CMAKE_BUILD_TYPE
  string(TOLOWER ${CMAKE_BUILD_TYPE} build_type)
  if (build_type STREQUAL debug)
    target_compile_definitions(exe1 PRIVATE DEBUG_BUILD)
  endif()

:manual:`Generator expressions <cmake-generator-expressions(7)>` should be
used instead to handle configuration-specific logic correctly, regardless of
the generator used.  For example:

.. code-block:: cmake

  # Works correctly for both single and multi-config generators
  target_compile_definitions(exe1 PRIVATE
    $<$<CONFIG:Debug>:DEBUG_BUILD>
  )

In the presence of :prop_tgt:`IMPORTED` targets, the content of
:prop_tgt:`MAP_IMPORTED_CONFIG_DEBUG <MAP_IMPORTED_CONFIG_<CONFIG>>` is also
accounted for by the above :genex:`$<CONFIG:Debug>` expression.


Case Sensitivity
----------------

:variable:`CMAKE_BUILD_TYPE` and :variable:`CMAKE_CONFIGURATION_TYPES` are
just like other variables in that any string comparisons made with their
values will be case-sensitive.  The :genex:`$<CONFIG>` generator expression also
preserves the casing of the configuration as set by the user or CMake defaults.
For example:

.. code-block:: cmake

  # NOTE: Don't use these patterns, they are for illustration purposes only.

  set(CMAKE_BUILD_TYPE Debug)
  if(CMAKE_BUILD_TYPE STREQUAL DEBUG)
    # ... will never get here, "Debug" != "DEBUG"
  endif()
  add_custom_target(print_config ALL
    # Prints "Config is Debug" in this single-config case
    COMMAND ${CMAKE_COMMAND} -E echo "Config is $<CONFIG>"
    VERBATIM
  )

  set(CMAKE_CONFIGURATION_TYPES Debug Release)
  if(DEBUG IN_LIST CMAKE_CONFIGURATION_TYPES)
    # ... will never get here, "Debug" != "DEBUG"
  endif()

In contrast, CMake treats the configuration type case-insensitively when
using it internally in places that modify behavior based on the configuration.
For example, the :genex:`$<CONFIG:Debug>` generator expression will evaluate to 1
for a configuration of not only ``Debug``, but also ``DEBUG``, ``debug`` or
even ``DeBuG``.  Therefore, you can specify configuration types in
:variable:`CMAKE_BUILD_TYPE` and :variable:`CMAKE_CONFIGURATION_TYPES` with
any mixture of upper and lowercase, although there are strong conventions
(see the next section).  If you must test the value in string comparisons,
always convert the value to upper or lowercase first and adjust the test
accordingly.

Default And Custom Configurations
---------------------------------

By default, CMake defines a number of standard configurations:

* ``Debug``
* ``Release``
* ``RelWithDebInfo``
* ``MinSizeRel``

In multi-config generators, the :variable:`CMAKE_CONFIGURATION_TYPES` variable
will be populated with (potentially a subset of) the above list by default,
unless overridden by the project or user.  The actual configuration used is
selected by the user at build time.

For single-config generators, the configuration is specified with the
:variable:`CMAKE_BUILD_TYPE` variable at configure time and cannot be changed
at build time.  The default value will often be none of the above standard
configurations and will instead be an empty string.  A common misunderstanding
is that this is the same as ``Debug``, but that is not the case.  Users should
always explicitly specify the build type instead to avoid this common problem.

The above standard configuration types provide reasonable behavior on most
platforms, but they can be extended to provide other types.  Each configuration
defines a set of compiler and linker flag variables for the language in use.
These variables follow the convention :variable:`CMAKE_<LANG>_FLAGS_<CONFIG>`,
where ``<CONFIG>`` is always the uppercase configuration name.  When defining
a custom configuration type, make sure these variables are set appropriately,
typically as cache variables.


Pseudo Targets
==============

Some target types do not represent outputs of the buildsystem, but only inputs
such as external dependencies, aliases or other non-build artifacts.  Pseudo
targets are not represented in the generated buildsystem.

.. _`Imported Targets`:

Imported Targets
----------------

An :prop_tgt:`IMPORTED` target represents a pre-existing dependency.  Usually
such targets are defined by an upstream package and should be treated as
immutable. After declaring an :prop_tgt:`IMPORTED` target one can adjust its
target properties by using the customary commands such as
:command:`target_compile_definitions`, :command:`target_include_directories`,
:command:`target_compile_options` or :command:`target_link_libraries` just like
with any other regular target.

:prop_tgt:`IMPORTED` targets may have the same usage requirement properties
populated as binary targets, such as
:prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES`,
:prop_tgt:`INTERFACE_COMPILE_DEFINITIONS`,
:prop_tgt:`INTERFACE_COMPILE_OPTIONS`,
:prop_tgt:`INTERFACE_LINK_LIBRARIES`, and
:prop_tgt:`INTERFACE_POSITION_INDEPENDENT_CODE`.

The :prop_tgt:`LOCATION` may also be read from an IMPORTED target, though there
is rarely reason to do so.  Commands such as :command:`add_custom_command` can
transparently use an :prop_tgt:`IMPORTED` :prop_tgt:`EXECUTABLE <TYPE>` target
as a ``COMMAND`` executable.

The scope of the definition of an :prop_tgt:`IMPORTED` target is the directory
where it was defined.  It may be accessed and used from subdirectories, but
not from parent directories or sibling directories.  The scope is similar to
the scope of a cmake variable.

It is also possible to define a ``GLOBAL`` :prop_tgt:`IMPORTED` target which is
accessible globally in the buildsystem.

See the :manual:`cmake-packages(7)` manual for more on creating packages
with :prop_tgt:`IMPORTED` targets.

.. _`Alias Targets`:

Alias Targets
-------------

An ``ALIAS`` target is a name which may be used interchangeably with
a binary target name in read-only contexts.  A primary use-case for ``ALIAS``
targets is for example or unit test executables accompanying a library, which
may be part of the same buildsystem or built separately based on user
configuration.

.. code-block:: cmake

  add_library(lib1 lib1.cpp)
  install(TARGETS lib1 EXPORT lib1Export ${dest_args})
  install(EXPORT lib1Export NAMESPACE Upstream:: ${other_args})

  add_library(Upstream::lib1 ALIAS lib1)

In another directory, we can link unconditionally to the ``Upstream::lib1``
target, which may be an :prop_tgt:`IMPORTED` target from a package, or an
``ALIAS`` target if built as part of the same buildsystem.

.. code-block:: cmake

  if (NOT TARGET Upstream::lib1)
    find_package(lib1 REQUIRED)
  endif()
  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 Upstream::lib1)

``ALIAS`` targets are not mutable, installable or exportable.  They are
entirely local to the buildsystem description.  A name can be tested for
whether it is an ``ALIAS`` name by reading the :prop_tgt:`ALIASED_TARGET`
property from it:

.. code-block:: cmake

  get_target_property(_aliased Upstream::lib1 ALIASED_TARGET)
  if(_aliased)
    message(STATUS "The name Upstream::lib1 is an ALIAS for ${_aliased}.")
  endif()

.. _`Interface Libraries`:

Interface Libraries
-------------------

An ``INTERFACE`` library target does not compile sources and does not
produce a library artifact on disk, so it has no :prop_tgt:`LOCATION`.

It may specify usage requirements such as
:prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES`,
:prop_tgt:`INTERFACE_COMPILE_DEFINITIONS`,
:prop_tgt:`INTERFACE_COMPILE_OPTIONS`,
:prop_tgt:`INTERFACE_LINK_LIBRARIES`,
:prop_tgt:`INTERFACE_SOURCES`,
and :prop_tgt:`INTERFACE_POSITION_INDEPENDENT_CODE`.
Only the ``INTERFACE`` modes of the :command:`target_include_directories`,
:command:`target_compile_definitions`, :command:`target_compile_options`,
:command:`target_sources`, and :command:`target_link_libraries` commands
may be used with ``INTERFACE`` libraries.

Since CMake 3.19, an ``INTERFACE`` library target may optionally contain
source files.  An interface library that contains source files will be
included as a build target in the generated buildsystem.  It does not
compile sources, but may contain custom commands to generate other sources.
Additionally, IDEs will show the source files as part of the target for
interactive reading and editing.

A primary use-case for ``INTERFACE`` libraries is header-only libraries.
Since CMake 3.23, header files may be associated with a library by adding
them to a header set using the :command:`target_sources` command:

.. code-block:: cmake

  add_library(Eigen INTERFACE)

  target_sources(Eigen PUBLIC
    FILE_SET HEADERS
      BASE_DIRS src
      FILES src/eigen.h src/vector.h src/matrix.h
  )

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 Eigen)

When we specify the ``FILE_SET`` here, the ``BASE_DIRS`` we define automatically
become include directories in the usage requirements for the target ``Eigen``.
The usage requirements from the target are consumed and used when compiling, but
have no effect on linking.

Another use-case is to employ an entirely target-focussed design for usage
requirements:

.. code-block:: cmake

  add_library(pic_on INTERFACE)
  set_property(TARGET pic_on PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE ON)
  add_library(pic_off INTERFACE)
  set_property(TARGET pic_off PROPERTY INTERFACE_POSITION_INDEPENDENT_CODE OFF)

  add_library(enable_rtti INTERFACE)
  target_compile_options(enable_rtti INTERFACE
    $<$<OR:$<COMPILER_ID:GNU>,$<COMPILER_ID:Clang>>:-rtti>
  )

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 pic_on enable_rtti)

This way, the build specification of ``exe1`` is expressed entirely as linked
targets, and the complexity of compiler-specific flags is encapsulated in an
``INTERFACE`` library target.

``INTERFACE`` libraries may be installed and exported. We can install the
default header set along with the target:

.. code-block:: cmake

  add_library(Eigen INTERFACE)

  target_sources(Eigen INTERFACE
    FILE_SET HEADERS
      BASE_DIRS src
      FILES src/eigen.h src/vector.h src/matrix.h
  )

  install(TARGETS Eigen EXPORT eigenExport
    FILE_SET HEADERS DESTINATION include/Eigen)
  install(EXPORT eigenExport NAMESPACE Upstream::
    DESTINATION lib/cmake/Eigen
  )

Here, the headers defined in the header set are installed to ``include/Eigen``.
The install destination automatically becomes an include directory that is a
usage requirement for consumers.

.. rubric:: 日本語訳注記

.. [#hint_for_framework_and_bundle_of_ios] 「`Frameworkとは＠Qiita <https://qiita.com/gdate/items/b49ef26824504bb61856#framework%E3%81%A8%E3%81%AF>`_」参照。
.. [#hint_for_usage_requirements] 「`CMake再入門メモ <https://zenn.dev/rjkuro/articles/054dab5b0e4f40#build-specification%E3%81%A8usage-requirement>`_」参照。
.. [#hint_for_transitive] 次々に移って行くこと。「等号の―性」（たとえば a = b で b = c ならば必ず a = c という性質）。
.. [#hint_for_build_specification] 「ビルドの仕様」とはターゲットAのビルドに必要な設定情報（ターゲット・プロパティ）、「利用要件」とはそのターゲットAを利用するターゲットB側で必要な設定情報（ターゲット・プロパティ）。利用要件のターゲット・プロパティはビルドの仕様のターゲット・プロパティに ``INTERFACE_`` という接頭辞を付けたもの。
