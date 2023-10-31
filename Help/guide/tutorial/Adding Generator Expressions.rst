ステップ４: ジェネレータ式を追加する
====================================

:manual:`ジェネレータ式 <cmake-generator-expressions(7)>` はビルドシステムの生成中に評価され、構成したビルドごとに固有の情報を生成します。

:manual:`ジェネレータ式 <cmake-generator-expressions(7)>` は :prop_tgt:`LINK_LIBRARIES`、:prop_tgt:`INCLUDE_DIRECTORIES`、:prop_tgt:`COMPILE_DEFINITIONS` などいろいろなターゲット・プロパティのコンテキストで使用できます。
さらに :command:`target_link_libraries`、:command:`target_include_directories`、:command:`target_compile_definitions` などのコマンドを使用して、これらのプロパティを設定できます。

:manual:`ジェネレータ式 <cmake-generator-expressions(7)>` は条件付きリンク、コンパイル時に使用する条件、条件付きインクルード・ディレクトリなどを有効にする際にも使えます。
これらの「条件」はビルド構成やターゲットのプロパティ、プラットフォームの情報、あるいはその他、コンパイラ等に問い合わせが可能な情報のことです。

:manual:`ジェネレータ式 <cmake-generator-expressions(7)>` には論理式、データ式、そして出力式など、さまざまな種類があります。

たとえば論理式は条件付き出力を生成する際に使用します。
もっとも基本的な論理式は ``0`` と ``1`` です。
他に、式 ``$<0:...>`` は空文字を返し、式 ``<1:...>`` は ``...`` の内容を返します。
これら二つの式はネストさせることもできます。

演習１ - ジェネレータ式を使ってコンパイラのワーニング・フラグを条件付きで追加する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`  のもっとも典型的な使い方は、プログラミング言語の最適化やワーニングのようなコンパイラ・フラグを条件付きで追加するというものです。
この方法のメリットは、コンパイラ・フラグを ``INTERFACE`` 型のライブラリなどのターゲットに関連付けることで、フラグを伝搬させることができるようになることです。

目標
----

ビルド時にコンパイラのワーニング・フラグ [#hint_for_warninig_flags]_ を追加する。ただしリリース版 [#hint_for_releasing]_ には追加しない。

参考情報
--------

* :manual:`cmake-generator-expressions(7)`
* :command:`cmake_minimum_required`
* :command:`set`
* :command:`target_compile_options`

編集するファイル
----------------

* ``CMakeLists.txt``

始める
------

``Step4/CMakeLists.txt`` を開いて ``TODO 1`` からh ``TODO 4`` まで進めて下さい。

まず、このプロジェクト最上位の ``CMakeLists.txt`` で :command:`cmake_minimum_required` コマンドを呼び出して、CMake の最低バージョンを ``3.15`` にセットします。
この演習では CMake バージョン 3.15 で導入されたジェネレータ式を使う予定です。

次に、このプロジェクト向けにコンパイラのワーニング・フラグを追加します。
このワーニング・フラグはコンパイラに依存するため、``COMPILE_LANG_AND_ID`` というジェネレータ式を使って、どのフラグをどのプログラミング言語とコンパイラに適用させるかを調節します。


ビルドと実行
------------

``Step4_build`` というディレクトリを新たに作成し、:manual:`cmake <cmake(1)>` コマンドまたは :manual:`cmake-gui <cmake-gui(1)>` を実行してプロジェクトを構成し、選択したビルド・ツールを使うか、またはビルド・ディレクトリから ``cmake --build .`` を実行してプロジェクトをビルドします。

.. code-block:: console

  mkdir Step4_build
  cd Step4_build
  cmake ../Step4
  cmake --build .

解決方法
--------

少なくとも CMake のバージョン ``3.15`` が必要であることを :command:`cmake_minimum_required` コマンドで指定します：

.. raw:: html

  <details><summary>TODO 1: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step5/CMakeLists.txt
  :caption: TODO 1: CMakeLists.txt
  :name: MathFunctions-CMakeLists.txt-minimum-required-step4
  :language: cmake
  :end-before: # set the project name and version

.. raw:: html

  </details>

次に、ワーニング・フラグは使用するコンパイラに依存するので、ビルドシステムでどのコンパイラを使用するのか確認します。
これはジェネレータ式の ``COMPILE_LANG_AND_ID`` を使います。
次のように、この式を評価した結果を格納する変数 ``gcc_like_cxx`` と ``msvc_cxx`` を用意します：

.. raw:: html

  <details><summary>TODO 2: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step5/CMakeLists.txt
  :caption: TODO 2: CMakeLists.txt
  :name: CMakeLists.txt-compile_lang_and_id
  :language: cmake
  :start-after: # the BUILD_INTERFACE genex
  :end-before: target_compile_options(tutorial_compiler_flags INTERFACE

.. raw:: html

  </details>

そして、このプロジェクトで使用するコンパイラのワーニング・フラグを追加します。
変数 ``gcc_like_cxx`` と ``msvc_cxx`` を使って、別のジェネレータ式を使い、これらの変数が TRUE の場合にのみ、それぞれのフラグを適用することもできます。
:command:`target_compile_options` コマンドを使い、これらのフラグを INTERFACE 型のライブラリに適用します。

.. raw:: html

  <details><summary>TODO 3: （クリックして答えを見る／隠す）</summary>

.. code-block:: cmake
  :caption: TODO 3: CMakeLists.txt
  :name: CMakeLists.txt-compile_flags

  target_compile_options(tutorial_compiler_flags INTERFACE
    "$<${gcc_like_cxx}:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>"
    "$<${msvc_cxx}:-W3>"
  )

.. raw:: html

  </details>

最後に、これらのワーニング・フラグはビルドでのみ使用したいと考えています。
プロジェクトをインストールしたユーザには、これらのフラグを継承すべきではありません。
そのため、``TODO3`` で適用したフラグを ``BUILD_INTERFACE`` という条件を評価するジェネレータ式でラップします。
以上の変更はつぎのようになります：

.. raw:: html

  <details><summary>TODO 4: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step5/CMakeLists.txt 
  :caption: TODO 4: CMakeLists.txt
  :name: CMakeLists.txt-target_compile_options-genex
  :language: cmake
  :start-after: set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
  :end-before: # configure a header file to pass some of the CMake settings

.. raw:: html

  </details>

.. rubric:: 日本語訳注記

.. [#hint_for_warninig_flags] 警告を表すメッセージを有効にするフラグなど。
.. [#hint_for_releasing] ビルドしたライブラリを利用する（リンクする）際にはフラグを追加しない。
