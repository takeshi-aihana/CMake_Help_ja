.. cmake-manual-description: CMake ビルドシステム・リファレンス

cmake-buildsystem(7)
********************

.. only:: html

   .. contents::

はじめに
========

CMake 系のビルドシステムは論理的な「**ターゲット**」の集まりとして構成されます。
これらのターゲットは、それぞれ一個の実行形式またはライブラリ、あるいは独自のコマンド列を実行するカスタム・ターゲットに相当します。
ターゲット間の依存関係はビルドシステムの中で構築されて、ビルドする順番や（``CMakeLists.txt`` の）変更に応じた再構成のルールを決定します。

.. _`Binary Targets`:

バイナリのターゲット
====================

実行形式とライブラリは、それぞれ :command:`add_executable` と :command:`add_library` のコマンドを使ってターゲットが定義されます。
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
~~~~~~~~~~~~~~~~

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
++++++++++++++++++++++

MacOS や iOS のフレームワーク [#hint_for_framework_and_bundle_of_ios]_ とそのバンドル [#hint_for_framework_and_bundle_of_ios]_ を生成する際、``SHARED`` ライブラリに :prop_tgt:`FRAMEWORK` というターゲット・プロパティが有効（ ``TRUE`` ）になっている場合があります。
一般的に、この ``FRAMEWORK`` というターゲット・プロパティが有効になったライブラリは、さらに :prop_tgt:`FRAMEWORK_VERSION` というターゲット・プロパティも有効になっているはずです。
このプロパティは通常、MacOS の慣例に倣って "A" の値がセットされます。
``MACOSX_FRAMEWORK_IDENTIFIER`` には  ``CFBundleIdentifier`` キーの値をセットして、バンドルを一意に識別します。

.. code-block:: cmake

  add_library(MyFramework SHARED MyFramework.cpp)
  set_target_properties(MyFramework PROPERTIES
    FRAMEWORK TRUE
    FRAMEWORK_VERSION A # バージョン "A" は macOS の慣例
    MACOSX_FRAMEWORK_IDENTIFIER org.cmake.MyFramework
  )

.. _`Object Libraries`:

オブジェクト・ライブラリ
~~~~~~~~~~~~~~~~~~~~~~~~

``OBJECT`` ライブラリという種類は、指定したソース・ファイルをコンパイルして生成したブジェクト・ファイルをアーカイブ化せずに集めたものです。
これらのオブジェクト・ファイルは :genex:`$<TARGET_OBJECTS:name>` という文法で、別のターゲットの入力ソースとして利用することができます。

これは ``OBJECT`` ライブラリの中身（オブジェクト・ファイル）を別のターゲットに提供する際に使用できる「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」の一つです：

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

ターゲット・プロパティの :prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES` や :prop_tgt:`INTERFACE_COMPILE_DEFINITIONS`、そして :prop_tgt:`INTERFACE_COMPILE_OPTIONS` のエントリは「利用要件」（*Usage Requirements* ） [#hint_for_build_specification]_ のプロパティです
（これらのプロパティには、利用者が正しくコンパイルしターゲットとリンクするために必要なエントリを指定します）。
バイナリのターゲットの場合は :command:`target_link_libraries` コマンドに指定した各ターゲットで、接頭子 ``INTERFACE_`` が付いたプロパティのエントリをそれぞれ使います：

.. code-block:: cmake

  set(srcs archive.cpp zip.cpp)
  if (LZMA_FOUND)
    list(APPEND srcs lzma.cpp)
  endif()
  add_library(archive SHARED ${srcs})
  if (LZMA_FOUND)
    # archive ライブラリのソースは -DBUILDING_WITH_LZMA でコンパイルされる
    target_compile_definitions(archive PRIVATE BUILDING_WITH_LZMA)
  endif()
  target_compile_definitions(archive INTERFACE USING_ARCHIVE_LIB)

  add_executable(consumer)
  # 実行形式の consumer を archive ライブラリにリンクして、その利用要件を使い切る
  # consumer のソースは -DUSING_ARCHIVE_LIB でコンパイルされる
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
  # archiveExtras ライブラリのソースは -DUSING_ARCHIVE_LIB と
  # -DUSING_SERIALIZATION_LIB でコンパイルされる

  add_executable(consumer consumer.cpp)
  # 実行形式 consumer のソースは -DUSING_ARCHIVE_LIB でコンパイルされる
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

利用要件は、依存関係から ``INTERFACE_`` 系のターゲット・プロパティを読み取り、そのエントリを依存先の ``INTERFACE_`` 系 **ではない** プロパティの最後に追加することによって伝搬していきます。
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
ライブラリは、その利用者が PIC としてビルドされることが期待されますが、その一方で実行形式は PIC としてビルドされないことが期待されるためエラーになります。

``lib1`` と ``lib2`` ライブラリの利用要件は「互換性」はありません。
一方は、その利用者が PIC としてビルドされることが期待されますが、もう一方は、その利用者が PIC としてビルドされないことが期待されています。
``exe2`` が両方のライブラリにリンクし利用要件が衝突しているため、CMake はエラーを出力します::

  CMake Error: The INTERFACE_POSITION_INDEPENDENT_CODE property of "lib2" does
  not agree with the value of POSITION_INDEPENDENT_CODE already determined
  for "exe2".

両ライブラリで「互換性」を保つには、:prop_tgt:`POSITION_INDEPENDENT_CODE` プロパティに伝搬する全ての依存関係上の伝搬元でセットした :prop_tgt:`INTERFACE_POSITION_INDEPENDENT_CODE` プロパティの値（論理型）を同じにする必要があります。

この「互換性のあるインタフェース」のプロパティ（利用要件）を、ターゲット・プロパティの :prop_tgt:`COMPATIBLE_INTERFACE_BOOL` のエントリ（論理型）として指定しておけば、他のターゲット・プロパティにも拡張できます。
ここで指定したプロパティはそれぞれ、利用者側のターゲットと、依存関係として伝搬する利用要件のプロパティ（``INTERFACE_`` の接頭子を持つプロパティ）との間で互換性があるようにして下さい。

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

論理型ではないプロパティも「互換性のあるインタフェース」の算出に加えることが可能です。
たとえば :prop_tgt:`COMPATIBLE_INTERFACE_STRING` プロパティに指定したエントリは「何も指定しない」にするか、またはすべての依存関係の間で同じ文字列と比較するかのどちらかにする必要があります。
これは、たとえばターゲットの利用要件によって複数ある互換性のないバージョンのライブラリがリンクしたくない場合に利用できます：

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

:prop_tgt:`COMPATIBLE_INTERFACE_NUMBER_MAX` というターゲット・プロパティのエントリは数値型として評価され、エントリの中で最大値を計算します：

.. code-block:: cmake

  add_library(lib1Version2 SHARED lib1_v2.cpp)
  set_property(TARGET lib1Version2 PROPERTY INTERFACE_CONTAINER_SIZE_REQUIRED 200)
  set_property(TARGET lib1Version2 APPEND PROPERTY
    COMPATIBLE_INTERFACE_NUMBER_MAX CONTAINER_SIZE_REQUIRED
  )

  add_library(lib1Version3 SHARED lib1_v3.cpp)
  set_property(TARGET lib1Version3 PROPERTY INTERFACE_CONTAINER_SIZE_REQUIRED 1000)

  add_executable(exe1 exe1.cpp)
  # CONTAINER_SIZE_REQUIRED は "200"
  target_link_libraries(exe1 lib1Version2)

  add_executable(exe2 exe2.cpp)
  # CONTAINER_SIZE_REQUIRED は "1000"
  target_link_libraries(exe2 lib1Version2 lib1Version3)

同様に :prop_tgt:`COMPATIBLE_INTERFACE_NUMBER_MIN` というターゲット・プロパティは依存関係から伝搬してきたプロパティの最小値を計算する際に使用できます。

このように計算された「互換性のあるインタフェース」のプロパティは「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使って依存先の利用者側で参照することが可能です。

ここで、依存先の利用者側に対し「互換性のあるインタフェース」のプロパティに指定したプロパティのリストは他のプロパティにセットしたリストと重複しないようにして下さい。

プロパティのデバッグ
--------------------

「ビルドの仕様」が依存関係で定義される場合があるため、ターゲットのソース・コードやビルドの仕様を設定する際に必要なコードが部分的に欠落していると、コードの推論が困難になることがあります。
:manual:`cmake(1)` コマンドは依存関係で定義されるようなプロパティの内容を詳しく出力するデバッグ機能を提供しています。
CMake 変数である :variable:`CMAKE_DEBUG_TARGET_PROPERTIES` のドキュメントにデバッグが可能なプロパティの一覧があります：

.. code-block:: cmake

  set(CMAKE_DEBUG_TARGET_PROPERTIES
    INCLUDE_DIRECTORIES
    COMPILE_DEFINITIONS
    POSITION_INDEPENDENT_CODE
    CONTAINER_SIZE_REQUIRED
    LIB_VERSION
  )
  add_executable(exe1 exe1.cpp)

ターゲット・プロパティの  :prop_tgt:`COMPATIBLE_INTERFACE_BOOL` や :prop_tgt:`COMPATIBLE_INTERFACE_STRING` にセットされたエントリの場合のデバッグ出力には、どのターゲットがプロパティをセットしたか、そしてプロパティを定義した他の依存関係が含まれています。
また :prop_tgt:`COMPATIBLE_INTERFACE_NUMBER_MAX` と :prop_tgt:`COMPATIBLE_INTERFACE_NUMBER_MIN` のターゲット・プロパティの場合だと、依存関係から伝搬してきたプロパティの値や、その値が新しい依存を決定するのかどうかがデバッグ出力として表示されます。


ジェネレータ式を使用したビルドの仕様
------------------------------------

ビルドの仕様を生成する際には「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」を使う場合があります。
たとえば「互換性のある」プロパティの値を計算する時に ``TARGET_PROPERTY`` の式を使って参照できます：

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

この例では、``exe1`` のソース・ファイルは ``-DCONTAINER_SIZE=200`` でコンパイルされます。

ジェネレータ式の ``TARGET_PROPERTY`` と ``TARGET_POLICY`` はライブラリを利用するターゲット ``exe1`` のコンテキストで評価されます。
これは、すなわち利用要件の仕様がライブラリを利用する側に基づいて異なる評価を受ける可能性があることを意味します： **※ 2023/11/17 翻訳停止（何を云っているのか全く分からない）**

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

The ``exe1`` executable will be compiled with ``-DLIB1_WITH_EXE``, while the ``shared_lib`` shared library will be compiled with ``-DLIB1_WITH_SHARED_LIB`` and ``-DCONSUMER_CMP0041_NEW``, because policy :policy:`CMP0041` is ``NEW`` at the point where the ``shared_lib`` target is created.

The ``BUILD_INTERFACE`` expression wraps requirements which are only used when consumed from a target in the same buildsystem, or when consumed from a target exported to the build directory using the :command:`export` command.
The ``INSTALL_INTERFACE`` expression wraps requirements which are only used when consumed from a target which has been installed and exported with the :command:`install(EXPORT)` command:

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

In this case, the ``exe1`` executable will be compiled with ``-DClimbingStats_FROM_BUILD_LOCATION``.
The exporting commands generate  :prop_tgt:`IMPORTED` targets with either the ``INSTALL_INTERFACE`` or the ``BUILD_INTERFACE`` omitted, and the ``*_INTERFACE`` marker stripped away.
A separate project consuming the ``ClimbingStats`` package would contain:

.. code-block:: cmake

  find_package(ClimbingStats REQUIRED)

  add_executable(Downstream main.cpp)
  target_link_libraries(Downstream Upstream::ClimbingStats)

Depending on whether the ``ClimbingStats`` package was used from the build location or the install location, the ``Downstream`` target would be compiled with either ``-DClimbingStats_FROM_BUILD_LOCATION`` or ``-DClimbingStats_FROM_INSTALL_LOCATION``.
For more about packages and exporting see the :manual:`cmake-packages(7)` manual.

.. _`Include Directories and Usage Requirements`:

Include Directories and Usage Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

成果物の出力
------------

:command:`add_library` と :command:`add_executable` コマンドが生成したビルドシステムのターゲットは「:ref:`バイナリのターゲット <Binary Targets>`」をビルドするルールを生成します。
ビルドするバイナリの正確な出力場所は、ビルド構成やリンクする際の依存関係などにより実際にビルドする時に決まります。
なお、ビルドしたバイナリの名前や出力場所は ``TARGET_FILE`` と ``TARGET_LINKER_FILE`` とそれらに関連する式を使って参照することは可能です。
ただし、これらの式は「:ref:`オブジェクト・ライブラリ <Object Libraries>`」では機能しません。これは、このライブラリをビルドする際に生成されるファイルが一つだけでないからです。

次のセクションで詳しく説明するように、ターゲットがビルドする成果物（*Artifacts*）は三種類あります。
これらの分類は DLL ベースのプラットフォームとそうではないプラットフォームの間では異なります。
ちなみに Cygwin を含む全ての Windows 系システムは DLL ベースのプラットフォームです。

.. _`Runtime Output Artifacts`:

ランタイム形式の成果物
~~~~~~~~~~~~~~~~~~~~~~

ビルドシステムが生成したターゲットが *ランタイム* 形式の場合の成果物は次のとおりです：

* :command:`add_executable` コマンドによって生成される実行可能なターゲットは、実行形式のファイル（たとえば ``.exe``）

* DLL ベースのプラットフォームの場合： ``SHARED`` オプションを指定した :command:`add_library` コマンドによって生成される共有ライブラリのターゲットは、実行形式のファイル（たとえば ``.dll``）

:prop_tgt:`RUNTIME_OUTPUT_DIRECTORY` や :prop_tgt:`RUNTIME_OUTPUT_NAME` といったターゲット・プロパティを使って、ビルドツリー内にランタイム形式の成果物が格納される場所や成果物の名前を変更できます。

.. _`Library Output Artifacts`:

ライブラリ形式の成果物
~~~~~~~~~~~~~~~~~~~~~~

ビルドシステムが生成したターゲットが *ライブラリ* 形式の場合の成果物は次のとおりです:

* ``MODULE`` オプションを指定した :command:`add_library` コマンドによって生成されるライブラリのターゲットは、ロード可能なモジュール・ファイル（たとえば、``.dll`` とか ``.so``）

* DLL ベースではないプラットフォームの場合： ``SHARED``  オプションを指定した :command:`add_library` コマンドによって生成される共有ライブラリのターゲットは、共有ライブラリのファイル（たとえば、``.so`` とか ``.dylib``）

:prop_tgt:`LIBRARY_OUTPUT_DIRECTORY` や :prop_tgt:`LIBRARY_OUTPUT_NAME` といったターゲット・プロパティを使って、ビルドツリー内にライブラリ形式の成果物が格納される場所や成果物の名前を変更できます。

.. _`Archive Output Artifacts`:

アーカイブ形式の成果物
~~~~~~~~~~~~~~~~~~~~~~

ビルドシステムが生成したターゲットが *アーカイブ* 形式の場合の成果物は次のとおりです：

*  ``STATIC`` オプションを指定した :command:`add_library` コマンドによって生成される静的ライブラリのターゲットは、静的ライブラリのファイル（たとえば  ``.lib`` とか ``.a``）

* DLL ベースのプラットフォームの場合： ``SHARED`` オプションを指定した :command:`add_library` コマンドによって生成される共有ライブラリのターゲットは、インポート・ライブラリのファイル（たとえば、``.lib``）。このファイルは、生成されたライブラリが少なくと一個のアンマネージドなシンボルを外部に公開している場合にのみ生成される。

* DLL ベースのプラットフォームの場合： :prop_tgt:`ENABLE_EXPORTS` というターゲット・プロパティがセットされている（``TRUE``） の時に :command:`add_executable` コマンドで生成される実行形式のターゲットは、インポート・ライブラリのファイル（たとえば ``.lib``）

* AIX 系のプラットフォームの場合： :prop_tgt:`ENABLE_EXPORTS` というターゲット・プロパティがセットされている（``TRUE``） の時に :command:`add_executable` コマンドで生成される実行形式のターゲットは、リンカ・インポートのファイル（たとえば  ``.imp``）

* macOS 系のプラットフォームの場合： :prop_tgt:`ENABLE_EXPORTS` というターゲット・プロパティがセットされている（``TRUE``） の時に、``SHARED`` オプションを指定した :command:`add_library` コマンドによって生成される共有ライブラリのターゲットは、リンカ・インポートのファイル（たとえば  ``.tbd``） 

:prop_tgt:`ARCHIVE_OUTPUT_DIRECTORY` や :prop_tgt:`ARCHIVE_OUTPUT_NAME` といったターゲット・プロパティを使って、ビルドツリー内にアーカイブ形式の成果物が格納される場所や成果物の名前を変更できます。

ディレクトリを特定するコマンド
------------------------------

:command:`target_include_directories`、:command:`target_compile_definitions`、そして :command:`target_compile_options` といったコマンドは、一度に１個のターゲットにだけ作用します。
これに対して :command:`add_compile_definitions`、:command:`add_compile_options`、そして :command:`include_directories` といったコマンドにも同様の機能がありますが、作用するのはターゲットではなくディレクトリです。

.. _`Build Configurations`:

ビルドの構成
============

生成した「**ビルドの構成** （Build Configurations）」により、``Release`` や ``Debug`` といった特定の種類のビルドの仕様が決まります。
ビルドの構成を指定する方法は、生成時に使用した :manual:`ジェネレータ <cmake-generators(7)>` に依存します。
たとえば :ref:`Makefile Generators` や :generator:`Ninja` のような一個の構成しか生成しないジェネレータの場合、ビルドの構成はその生成時に CMake の変数 :variable:`CMAKE_BUILD_TYPE` の値で決まります。
対して :ref:`Visual Studio <Visual Studio Generators>` や :generator:`Xcode`、そして :generator:`Ninja Multi-Config` のような複数の構成を扱うジェネレータの場合、ユーザがビルド時に構成を選択するので、:variable:`CMAKE_BUILD_TYPE` の値は無視されます。
後者のジェネレータでは、CMake 変数の :variable:`CMAKE_CONFIGURATION_TYPES` で *利用可能な* 設定一式を指定できますが、実際に使用する設定はビルド時までわかりません。
この違いは誤解されることが多く、次のような問題のあるコードにつながります：

.. code-block:: cmake

  # 注意: この操作は複数の構成を扱うジェネレータでは間違い
  #      （そのようなジェネレータは基本的に CMAKE_BUILD_TYPE を利用しないので）
  string(TOLOWER ${CMAKE_BUILD_TYPE} build_type)
  if (build_type STREQUAL debug)
    target_compile_definitions(exe1 PRIVATE DEBUG_BUILD)
  endif()

このような場合、使用するジェネレータにかかわらず、構成に固有の仕様を正しく処理させるために CMake 変数ではなく、:manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を使用して下さい。
たとえば：

.. code-block:: cmake

  # このようにするとジェネレータの種類に依らず正しく動作する
  target_compile_definitions(exe1 PRIVATE
    $<$<CONFIG:Debug>:DEBUG_BUILD>
  )

:prop_tgt:`IMPORTED` なターゲットが存在する場合、:prop_tgt:`MAP_IMPORTED_CONFIG_DEBUG <MAP_IMPORTED_CONFIG_<CONFIG>>` というターゲット・プロパティはジェネレータ式の :genex:`$<CONFIG:Debug>`  の評価も考慮されます。

大小文字を区別する
------------------

CMake 変数の :variable:`CMAKE_BUILD_TYPE` と :variable:`CMAKE_CONFIGURATION_TYPES` は、これらの値を使った文字列比較では大文字と小文字が区別されるという点で、扱い方は他の変数と同じです。
ジェネレータ式の :genex:`$<CONFIG>` は、ユーザまたは CMake のデフォルトによって指定されたビルドの構成での大文字/小文字も保持します。
たとえば：

.. code-block:: cmake

  # 注意：実際に以下のような使い方はしないこと（これは便宜上、問題を説明するために使用しているだけ）

  set(CMAKE_BUILD_TYPE Debug)
  if(CMAKE_BUILD_TYPE STREQUAL DEBUG)
    # ... "Debug" != "DEBUG" なのでマッチしない
  endif()
  add_custom_target(print_config ALL
    # "Config is Debug" と出力するカスタム・ターゲット
  COMMAND ${CMAKE_COMMAND} -E echo "Config is $<CONFIG>"
    VERBATIM
  )

  set(CMAKE_CONFIGURATION_TYPES Debug Release)
  if(DEBUG IN_LIST CMAKE_CONFIGURATION_TYPES)
    # ... "Debug" != "DEBUG" なのでマッチしない
  endif()

それに対して、CMake はビルドの構成に応じて処理を変更させる場所で、構成の種類を内部で参照する際は大文字/小文字を区別せずに扱います。
たとえば、ジェネレータ式の :genex:`$<CONFIG:Debug>` は ``Debug`` だけでなく ``DEBUG`` や ``debug``、さらには ``DeBuG`` であっても同じ様に 1 と評価します。
そのため、CMake としては「厳格なルール」（次のセクションを参考のこと）はあるものの、CMake 変数の :variable:`CMAKE_BUILD_TYPE` と :variable:`CMAKE_CONFIGURATION_TYPES` では大文字と小文字を任意に混ぜて指定することも可能になっています。
もし文字列比較で変数の値をテストする場合は、必ず最初に値を大文字または小文字に変換し、それに応じてテストの方法も調整してから行うようにして下さい。

デフォルトの構成とカスタマイズした構成
--------------------------------------

CMake はデフォルトで標準的なビルドの構成をいくつか定義しています：

* ``Debug``
* ``Release``
* ``RelWithDebInfo``
* ``MinSizeRel``

複数の構成を扱うジェネレータでは、ユーザやプロジェクトによって上書きされない限り、デフォルトで CMake 変数の :variable:`CMAKE_CONFIGURATION_TYPES` の値に上記の定義（実際にはそのサブセット）がセットされます。
実際に使用するビルドの構成はビルド時にユーザが指定します。

一個の構成しか生成しないジェネレータの場合、ビルドの構成はその生成時に CMake の変数 :variable:`CMAKE_BUILD_TYPE` の値で決まり、ビルド時に構成を変更することはできません。
デフォルトで、このCMake 変数には、上記の標準的な構成ではなく、空の文字列がセットされます。
よくある誤解として、これが ``Debug`` と同じであるとするものですが、それは間違いです。
この問題を回避するため、ユーザは常にビルドの構成を明示的に指定する必要があります。

上記の標準的なビルドの構成は、ほとんどのプラットフォームで適切に動作しますが、他の構成を提供するように拡張することが可能です。
各構成は、使用しているプログラミング言語に対するコンパイラ・フラグとリンカ・フラグを変数として定義します。
これらの変数は :variable:`CMAKE_<LANG>_FLAGS_<CONFIG>` というスタイルを持ちます（構成の名前を表す ``<CONFIG>`` は常に大文字にすること）。
独自にカスタマイズした構成を定義する際は、これらの変数が（通常はキャッシュ変数として） 適切に設定されていることを確認して下さい。

疑似ターゲット
==============

ビルドの構成の中には、ビルドシステムのログを表示しないで、代わりに外部との依存関係、別名（エイリアス）、またはビルドの対象外の成果物の情報だけ表示するものがあります。
「**疑似ターゲット**」は、ビルドシステムの生成中は何も表示されません。

.. _`Imported Targets`:

IMPORTED なターゲット
---------------------

:prop_tgt:`IMPORTED` なターゲットは既存の依存関係を表します（？）。
通常、このようなターゲットは上流のパッケージによって定義され変更不可として扱われる必要があります。
:prop_tgt:`IMPORTED` なターゲットは、他の通常のターゲットと同様に、:command:`target_compile_definitions`、:command:`target_include_directories`、:command:`target_compile_options`、あるいは :command:`target_link_libraries` といったコマンドを使って、そのターゲット・プロパティを変更できます。

:prop_tgt:`IMPORTED` なターゲットには、:prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES`、:prop_tgt:`INTERFACE_COMPILE_DEFINITIONS`、:prop_tgt:`INTERFACE_COMPILE_OPTIONS`、:prop_tgt:`INTERFACE_LINK_LIBRARIES`、:prop_tgt:`INTERFACE_POSITION_INDEPENDENT_CODE` など、「:ref:`バイナリのターゲット <Binary Targets>`」と同じ「:ref:`利用要件 <Target Usage Requirements>`」（*Usage Requirements*） [#hint_for_build_specification]_ のプロパティが設定されている場合があります。

ターゲット・プロパティの :prop_tgt:`LOCATION` も :prop_tgt:`IMPORTED` なターゲットから読みとられる場合がありますが、必ず読み取らなければならないという訳ではありません。
:command:`add_custom_command` などのコマンドでは、:prop_tgt:`IMPORTED` で :prop_tgt:`EXECUTABLE <TYPE>` なターゲットを実行形式 ``COMMAND`` として透過的に利用できます。

:prop_tgt:`IMPORTED` なターゲットの定義は、それが定義されたディレクトリが有効なスコープです。
スコープのサブディレクトリからアクセスしたり利用することも可能ですが、スコープの親ディレクトリや兄弟ディレクトリからはアクセスできません。
このスコープは CMake 変数のスコープと似ています。

さらに、ビルドシステムからグローバルにアクセスが可能な ``GLOBAL`` で :prop_tgt:`IMPORTED` なターゲットを定義することも可能です。

:prop_tgt:`IMPORTED` なターゲットからパッケージを作成する方法について詳細は :manual:`cmake-packages(7)` のマニュアルを参照して下さい。

.. _`Alias Targets`:

ALIAS ターゲット
----------------

``ALIAS`` ターゲットは、読み取り専用のコンテキストで「:ref:`バイナリのターゲット <Binary Targets>`」の名前と同じように扱うことができる別の名前です。
この ``ALIAS`` ターゲットの主な使い途としては、たとえばライブラリと一緒に行う実行形式の単体テストがあります。このテストは、同じビルドシステムの一部であったり、あるいはユーザが生成した構成に基づいて別々にビルドされる場合があります。

.. code-block:: cmake

  add_library(lib1 lib1.cpp)
  install(TARGETS lib1 EXPORT lib1Export ${dest_args})
  install(EXPORT lib1Export NAMESPACE Upstream:: ${other_args})

  add_library(Upstream::lib1 ALIAS lib1)

この例では、別のディレクトリで ``Upstream::lib1`` というターゲットに無条件でリンクできます。これは、任意のパッケージからの :prop_tgt:`IMPORTED` なターゲットであるかもしれないし、あるいは同じビルドシステムの一部としてビルドされた場合の ``ALIAS`` ターゲットの可能性があります。

.. code-block:: cmake

  if (NOT TARGET Upstream::lib1)
    find_package(lib1 REQUIRED)
  endif()
  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 Upstream::lib1)

``ALIAS`` ターゲットは変更不可で、インストールもエキスポートもできません。
これらのターゲットはビルドシステムの記述に対して完全にローカルな扱いです。
:prop_tgt:`ALIASED_TARGET` というターゲット・プロパティの値から、ターゲット名が ``ALIAS`` であるかテストできます：

.. code-block:: cmake

  get_target_property(_aliased Upstream::lib1 ALIASED_TARGET)
  if(_aliased)
    message(STATUS "The name Upstream::lib1 is an ALIAS for ${_aliased}.")
  endif()

.. _`Interface Libraries`:

INTERFACE ライブラリなターゲット
--------------------------------

``INTERFACE`` ライブラリなターゲットはソースをコンパイルせず成果物となるファイルを出力しないため、:prop_tgt:`LOCATION` プロパティは持ちません。

ターゲット・プロパティの :prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES`、:prop_tgt:`INTERFACE_COMPILE_DEFINITIONS`、:prop_tgt:`INTERFACE_COMPILE_OPTIONS`、:prop_tgt:`INTERFACE_LINK_LIBRARIES`、:prop_tgt:`INTERFACE_SOURCES`、そして :prop_tgt:`INTERFACE_POSITION_INDEPENDENT_CODE` などの「:ref:`利用要件 <Target Usage Requirements>`」（*Usage Requirements*） [#hint_for_build_specification]_ を指定する場合があります。
It may specify usage requirements such as :prop_tgt:`INTERFACE_INCLUDE_DIRECTORIES`, :prop_tgt:`INTERFACE_COMPILE_DEFINITIONS`, :prop_tgt:`INTERFACE_COMPILE_OPTIONS`, :prop_tgt:`INTERFACE_LINK_LIBRARIES`, :prop_tgt:`INTERFACE_SOURCES`, and :prop_tgt:`INTERFACE_POSITION_INDEPENDENT_CODE`.
Only the ``INTERFACE`` modes of the :command:`target_include_directories`, :command:`target_compile_definitions`, :command:`target_compile_options`, :command:`target_sources`, and :command:`target_link_libraries` commands may be used with ``INTERFACE`` libraries.

Since CMake 3.19, an ``INTERFACE`` library target may optionally contain source files.
An interface library that contains source files will be included as a build target in the generated buildsystem.
It does not compile sources, but may contain custom commands to generate other sources.
Additionally, IDEs will show the source files as part of the target for interactive reading and editing.

A primary use-case for ``INTERFACE`` libraries is header-only libraries.
Since CMake 3.23, header files may be associated with a library by adding them to a header set using the :command:`target_sources` command:

.. code-block:: cmake

  add_library(Eigen INTERFACE)

  target_sources(Eigen PUBLIC
    FILE_SET HEADERS
      BASE_DIRS src
      FILES src/eigen.h src/vector.h src/matrix.h
  )

  add_executable(exe1 exe1.cpp)
  target_link_libraries(exe1 Eigen)

When we specify the ``FILE_SET`` here, the ``BASE_DIRS`` we define automatically become include directories in the usage requirements for the target ``Eigen``.
The usage requirements from the target are consumed and used when compiling, but have no effect on linking.

Another use-case is to employ an entirely target-focussed design for usage requirements:

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

This way, the build specification of ``exe1`` is expressed entirely as linked targets, and the complexity of compiler-specific flags is encapsulated in an ``INTERFACE`` library target.

``INTERFACE`` libraries may be installed and exported.
We can install the default header set along with the target:

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
The install destination automatically becomes an include directory that is a usage requirement for consumers.

.. rubric:: 日本語訳注記

.. [#hint_for_framework_and_bundle_of_ios] 「`Frameworkとは＠Qiita <https://qiita.com/gdate/items/b49ef26824504bb61856#framework%E3%81%A8%E3%81%AF>`_」参照。
.. [#hint_for_usage_requirements] 「`CMake再入門メモ <https://zenn.dev/rjkuro/articles/054dab5b0e4f40#build-specification%E3%81%A8usage-requirement>`_」参照。
.. [#hint_for_transitive] 次々に移って行くこと。「等号の―性」（たとえば a = b で b = c ならば必ず a = c という性質）。
.. [#hint_for_build_specification] 「ビルドの仕様」とはターゲットAのビルドに必要な設定情報（ターゲット・プロパティ）、「利用要件」とはそのターゲットAを利用するターゲットB側で必要な設定情報（ターゲット・プロパティ）。利用要件のターゲット・プロパティはビルドの仕様のターゲット・プロパティに ``INTERFACE_`` という接頭辞を付けたもの。
