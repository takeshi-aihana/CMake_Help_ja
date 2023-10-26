ステップ３: ライブラリの使用要件を追加する
==========================================

演習１ - ライブラリの使用要件を追加する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ターゲット・パラメータの「:ref:`使用要件 <Target Usage Requirements>`」（*Usage Requirements*） を使うと、ライブラリや実行形式のリンクと include 行をより適切に制御できると共に、CMake 内で「遷移する」ターゲットのプロパティをより細かく制御できるようになります。
これを活用する主なコマンドは：

* :command:`target_compile_definitions`
* :command:`target_compile_options`
* :command:`target_include_directories`
* :command:`target_link_directories`
* :command:`target_link_options`
* :command:`target_precompile_headers`
* :command:`target_sources`


目標
----

ライブラリに「使用要件」を追加する。

参考情報
--------

* :variable:`CMAKE_CURRENT_SOURCE_DIR`

編集するファイル
----------------

* ``MathFunctions/CMakeLists.txt``
* ``CMakeLists.txt``

始める
------

この演習では、CMake の最新のアプローチを利用するために 「:guide:`tutorial/Adding a Library`」で演習したコードをリファクタリングします。
ライブラリに独自の「使用要件」を定義させて、必要に応じて他のターゲットに渡せるようにします。
この時 ``MathFunctions`` は必要な include ディレクトリそのものを指定します。
次に、使用するターゲットの ``Tutorial`` は ``MathFunctions`` にリンクするだけで、追加の include ディレクトリについて心配する必要はありません。

出発点は ``Step3`` ディレクトリにあるソース・ファイルです。
この演習では ``TODO 1`` から始めて ``TODO 3`` まで進めて下さい。

まず ``MathFunctions/CMakeLists`` に :command:`target_include_directories` コマンドを追加します。
変数の :variable:`CMAKE_CURRENT_SOURCE_DIR` は、現在作業しているソース・ファイルがあるパス名です。

それからプロジェクト最上位の ``CMakeLists.txt`` にある :command:`target_include_directories` コマンドの呼び出しを修正して短くします。

ビルドと実行
-------------

あらたに ``Step3_build`` ディレクトリを作成し、:manual:`cmake <cmake(1)>` コマンドまたは :manual:`cmake-gui <cmake-gui(1)>` を実行してプロジェクトを構成し、選択したビルド・ツールを使う、またはビルド・ディレクトリから :option:`cmake --build . <cmake --build>` を実行してプロジェクトをビルドします。
この一連のコマンドライン操作は次のようになります：

.. code-block:: console

  mkdir Step3_build
  cd Step3_build
  cmake ../Step3
  cmake --build .

次に、新しくビルドした実行形式の ``Tutorial`` を使って期待通り動作するか確認して下さい。

解決方法
--------

前のステップのコードを修正して、CMake の最新のアプローチである「使用要件」を使ってみることにしましょう。

We want to state that anybody linking to ``MathFunctions`` needs to include the current source directory, while ``MathFunctions`` itself doesn't.
This can be expressed with an ``INTERFACE`` usage requirement.
Remember ``INTERFACE`` means things that consumers require but the producer doesn't.

At the end of ``MathFunctions/CMakeLists.txt``, use :command:`target_include_directories` with the ``INTERFACE`` keyword, as follows:

.. raw:: html

  <details><summary>TODO 1: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step4/MathFunctions/CMakeLists.txt
  :caption: TODO 1: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-target_include_directories-INTERFACE
  :language: cmake
  :start-after: # to find MathFunctions.h
  :end-before: # should we use our own

.. raw:: html

  </details>

Now that we've specified usage requirements for ``MathFunctions`` we can safely remove our uses of the ``EXTRA_INCLUDES`` variable from the top-level ``CMakeLists.txt``.

Remove this line:

.. raw:: html

  <details><summary>TODO 2: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/CMakeLists.txt
  :caption: TODO 2: CMakeLists.txt
  :name: CMakeLists.txt-remove-EXTRA_INCLUDES
  :language: cmake
  :start-after: add_subdirectory(MathFunctions)
  :end-before: # add the executable

.. raw:: html

  </details>

And remove ``EXTRA_INCLUDES`` from ``target_include_directories``:

.. raw:: html

  <details><summary>TODO 3: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step4/CMakeLists.txt
  :caption: TODO 3: CMakeLists.txt
  :name: CMakeLists.txt-target_include_directories-remove-EXTRA_INCLUDES
  :language: cmake
  :start-after: # so that we will find TutorialConfig.h

.. raw:: html

  </details>

Notice that with this technique, the only thing our executable target does to use our library is call :command:`target_link_libraries` with the name of the library target.
In larger projects, the classic method of specifying library dependencies manually becomes very complicated very quickly.

演習２ - インタフェース・ライブラリに C++ 標準を適用する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that we have switched our code to a more modern approach, let's demonstrate a modern technique to set properties to multiple targets.

Let's refactor our existing code to use an ``INTERFACE`` library.
We will use that library in the next step to demonstrate a common use for :manual:`generator expressions <cmake-generator-expressions(7)>`.

目標
----

Add an ``INTERFACE`` library target to specify the required C++ standard.

参考情報
--------

* :command:`add_library`
* :command:`target_compile_features`
* :command:`target_link_libraries`

編集するファイル
----------------

* ``CMakeLists.txt``
* ``MathFunctions/CMakeLists.txt``

始める
------

In this exercise, we will refactor our code to use an ``INTERFACE`` library to specify the C++ standard.

Start this exercise from what we left at the end of Step3 exercise 1.
You will have to complete ``TODO 4`` through ``TODO 7``.

Start by editing the top level ``CMakeLists.txt`` file.
Construct an ``INTERFACE`` library target called ``tutorial_compiler_flags`` and specify ``cxx_std_11`` as a target compiler feature.

Modify ``CMakeLists.txt`` and ``MathFunctions/CMakeLists.txt`` so that all targets have a :command:`target_link_libraries` call to ``tutorial_compiler_flags``.

ビルドと実行
------------

Since we have our build directory already configured from Exercise 1, simply rebuild our code by calling the following:

.. code-block:: console

  cd Step3_build
  cmake --build .

Next, use the newly built ``Tutorial`` and verify that it is working as expected.

解決方法
--------

Let's update our code from the previous step to use interface libraries to set our C++ requirements.

To start, we need to remove the two :command:`set` calls on the variables :variable:`CMAKE_CXX_STANDARD` and :variable:`CMAKE_CXX_STANDARD_REQUIRED`.
The specific lines to remove are as follows:

.. literalinclude:: Step3/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-CXX_STANDARD-variable-remove
  :language: cmake
  :start-after: # specify the C++ standard
  :end-before: # configure a header file

Next, we need to create an interface library, ``tutorial_compiler_flags``.
And then use :command:`target_compile_features` to add the compiler feature ``cxx_std_11``.


.. raw:: html

  <details><summary>TODO 4: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step4/CMakeLists.txt
  :caption: TODO 4: CMakeLists.txt
  :name: CMakeLists.txt-cxx_std-feature
  :language: cmake
  :start-after: # specify the C++ standard
  :end-before: # TODO 2: Create helper

.. raw:: html

  </details>

Finally, with our interface library set up, we need to link our executable ``Tutorial``, our ``SqrtLibrary`` library and our ``MathFunctions`` library to our new ``tutorial_compiler_flags`` library.
Respectively, the code will look like this:

.. raw:: html

  <details><summary>TODO 5: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step4/CMakeLists.txt
  :caption: TODO 5: CMakeLists.txt
  :name: CMakeLists.txt-target_link_libraries-step4
  :language: cmake
  :start-after: add_executable(Tutorial tutorial.cxx)
  :end-before: # add the binary tree to the search path for include file

.. raw:: html

  </details>

this:

.. raw:: html

  <details><summary>TODO 6: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step4/MathFunctions/CMakeLists.txt
  :caption: TODO 6: MathFunctions/CMakeLists.txt
  :name: MathFunctions-CMakeLists.txt-target_link_libraries-step4
  :language: cmake
  :start-after: # link SqrtLibrary to tutorial_compiler_flags
  :end-before: target_link_libraries(MathFunctions

.. raw:: html

  </details>

and this:

.. raw:: html

  <details><summary>TODO 7: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step4/MathFunctions/CMakeLists.txt
  :caption: TODO 7: MathFunctions/CMakeLists.txt
  :name: MathFunctions-SqrtLibrary-target_link_libraries-step4
  :language: cmake
  :start-after: # link MathFunctions to tutorial_compiler_flags

.. raw:: html

  </details>


With this, all of our code still requires C++ 11 to build.
Notice  though that with this method, it gives us the ability to be specific about which targets get specific requirements.
In addition, we create a single source of truth in our interface library.
