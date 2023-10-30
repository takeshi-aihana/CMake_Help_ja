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

演習１ - ジェネレータ式を使ってコンパイラのワーニング・フラグを追加する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`  のもっとも典型的な使い方は、プログラミング言語の最適化やワーニングのようなコンパイラ・フラグを条件付きで追加するというものです。
この方法のメリットは、コンパイラ・フラグを ``INTERFACE`` 型のライブラリなどのターゲットに関連付けることで、フラグを伝搬させることができるようになることです。

目標
----

ビルド時にワーニング・フラグ [#hint_for_warninig_flags]_ を追加する。ただしリリース版には追加しない。

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

Open the file ``Step4/CMakeLists.txt`` and complete ``TODO 1`` through ``TODO 4``.

First, in the top level ``CMakeLists.txt`` file, we need to set the :command:`cmake_minimum_required` to ``3.15``.
In this exercise we are going to use a generator expression which was introduced in CMake 3.15.

Next we add the desired compiler warning flags that we want for our project.
As warning flags vary based on the compiler, we use the ``COMPILE_LANG_AND_ID`` generator expression to control which flags to apply given a language and a set of compiler ids.

ビルドと実行
------------

Make a new directory called ``Step4_build``, run the :manual:`cmake <cmake(1)>` executable or the :manual:`cmake-gui <cmake-gui(1)>` to configure the project and then build it with your chosen build tool or by using ``cmake --build .`` from the build directory.

.. code-block:: console

  mkdir Step4_build
  cd Step4_build
  cmake ../Step4
  cmake --build .

解決方法
--------

Update the :command:`cmake_minimum_required` to require at least CMake version ``3.15``:

.. raw:: html

  <details><summary>TODO 1: Click to show/hide answer</summary>

.. literalinclude:: Step5/CMakeLists.txt
  :caption: TODO 1: CMakeLists.txt
  :name: MathFunctions-CMakeLists.txt-minimum-required-step4
  :language: cmake
  :end-before: # set the project name and version

.. raw:: html

  </details>

Next we determine which compiler our system is currently using to build since warning flags vary based on the compiler we use.
This is done with the ``COMPILE_LANG_AND_ID`` generator expression.
We set the result in the variables ``gcc_like_cxx`` and ``msvc_cxx`` as follows:

.. raw:: html

  <details><summary>TODO 2: Click to show/hide answer</summary>

.. literalinclude:: Step5/CMakeLists.txt
  :caption: TODO 2: CMakeLists.txt
  :name: CMakeLists.txt-compile_lang_and_id
  :language: cmake
  :start-after: # the BUILD_INTERFACE genex
  :end-before: target_compile_options(tutorial_compiler_flags INTERFACE

.. raw:: html

  </details>

Next we add the desired compiler warning flags that we want for our project.
Using our variables ``gcc_like_cxx`` and ``msvc_cxx``, we can use another generator expression to apply the respective flags only when the variables are true.
We use :command:`target_compile_options` to apply these flags to our interface library.

.. raw:: html

  <details><summary>TODO 3: Click to show/hide answer</summary>

.. code-block:: cmake
  :caption: TODO 3: CMakeLists.txt
  :name: CMakeLists.txt-compile_flags

  target_compile_options(tutorial_compiler_flags INTERFACE
    "$<${gcc_like_cxx}:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>"
    "$<${msvc_cxx}:-W3>"
  )

.. raw:: html

  </details>

Lastly, we only want these warning flags to be used during builds.
Consumers of our installed project should not inherit our warning flags.
To specify this, we wrap our flags from TODO 3 in a generator expression using the ``BUILD_INTERFACE`` condition.
The resulting full code looks like the following:

.. raw:: html

  <details><summary>TODO 4: Click to show/hide answer</summary>

.. literalinclude:: Step5/CMakeLists.txt 
  :caption: TODO 4: CMakeLists.txt
  :name: CMakeLists.txt-target_compile_options-genex
  :language: cmake
  :start-after: set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
  :end-before: # configure a header file to pass some of the CMake settings

.. raw:: html

  </details>

.. rubric:: 日本語訳注記

..  [#hint_for_warninig_flags]  ワーニングレベルをエラーとして扱うフラグのこと。
