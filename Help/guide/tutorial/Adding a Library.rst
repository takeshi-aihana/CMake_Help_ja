ステップ２: ライブラリを追加する
================================

ここまで CMake を使って基本的なプロジェクトを作成してビルドする方法をみてきました。
このステップでは、プロジェクトでライブラリを作成し利用する方法について学習します。
また、そのライブラリの利用をオプションにする方法についても説明します。

演習１ - ライブラリを作成する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CMake でライブラリを追加するには、:command:`add_library` コマンドを使ってライブラリを構成しているソース・ファイルを指定します。

プロジェクトは一つのディレクトリにソース・ファイルをまとめて格納する他に、複数のサブディレクトリに分けて格納して構造化することができます。
このような場合、ライブラリ専用のサブディレクトリを作成します。
ここで、新しい ``CMakeLists.txt`` ファイルと、ソース・ファイルをいくつか追加してみます。
プロジェクト最上位の ``CMakeLists.txt`` ファイルの中に、:command:`add_subdirectory` コマンドを使ってビルドするサブディレクトリを追加します。

ライブラリを生成したら、:command:`target_include_directories` と :command:`target_link_libraries` のコマンドを使ってターゲットの実行形式にリンクします。

目標
----

ライブラリを一個追加してリンクする。

参考情報
--------

* :command:`add_library`
* :command:`add_subdirectory`
* :command:`target_include_directories`
* :command:`target_link_libraries`
* :variable:`PROJECT_SOURCE_DIR`

編集するファイル
----------------

* ``CMakeLists.txt``
* ``tutorial.cxx``
* ``MathFunctions/CMakeLists.txt``

始める
------

この演習では数値の平方根を計算する独自の実装を持たライブラリをプロジェクトに追加します。
そして実行形式は、コンパイラが提供する標準関数ではなく、このライブラリを使うようにします。

このチュートリアルでは ``MathFunctions`` というサブディレクトリの中にライブラリを配置します。
このサブディレクトリには既にヘッダファイルの ``MathFunctions.h`` と ``mysqrt.h`` が格納されています。
対応するソース・ファイルは、それぞれ ``MathFunctions.cxx`` と ``mysqrt.cxx`` です。
ただし、これらのファイルは変更はしません。
``mysqrt.cxx`` には ``mysqrt`` という関数があり、これはコンパイラが提供する標準の ``sqrt`` 関数と同様の機能が実装されています。
``MathFunctions.cxx`` には ``squrt`` の実装の詳細を隠す役割を持つ ``sqrt`` という関数が実装されています。

``Help/guide/tutorial/Step2`` ディレクトリへ移動して、``TODO 1`` から始めて ``TODO 6`` まで進めて下さい。

まず ``MathFunctions`` サブディレクトリにある ``CMakeLists.txt`` の一行を埋めます。

次にプロジェクト最上位にある ``CMakeLists.txt`` を編集します。

最後に新しく用意した ``MathFunctions`` ライブラリを ``tutorial.cxx`` の中から利用する実装を追加します。

ビルドと実行
-------------

:manual:`cmake  <cmake(1)>` コマンドか、 :manual:`cmake-gui <cmake-gui(1)>` を起動してプロジェクトを構成し、それから指定したビルド・ツールでプロジェクトをビルドします。

コマンドラインでは以下のようになります：

.. code-block:: console

  mkdir Step2_build
  cd Step2_build
  cmake ../Step2
  cmake --build .

ビルドした実行形式の ``Tutorial`` を使い、正しく平方根が計算されることを確認してみて下さい。

解決方法
--------

``MathFunctions`` ディレクトリの``CMakeLists.txt`` ファイルの中で、:command:`add_library` コマンドを使って ``MathFunctions`` というライブラリのターゲットを作成します。
ライブラリのソース・ファイルを :command:`add_library` コマンドの引数として渡します。
たとえば次のようになります：

.. raw:: html

  <details><summary>TODO 1: （クリックして答えを見る／隠す）</summary>

.. code-block:: cmake
  :caption: TODO 1: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-add_library

  add_library(MathFunctions MathFunctions.cxx mysqrt.cxx)

.. raw:: html

  </details>

プロジェクト最上位にある ``CMakeLists.txt`` ファイルの中で :command:`add_subdirectory` コマンドの呼び出しを追加すると、そのライブラリがビルドされるようになります。

.. raw:: html

  <details><summary>TODO 2: （クリックして答えを見る／隠す）</summary>

.. code-block:: cmake
  :caption: TODO 2: CMakeLists.txt
  :name: CMakeLists.txt-add_subdirectory

  add_subdirectory(MathFunctions)

.. raw:: html

  </details>

次に :command:`target_link_libraries` コマンドを使って、新しいライブラリ ``MathFunctions`` と実行形式 ``Tutorial`` をリンクします。

.. raw:: html

  <details><summary>TODO 3: （クリックして答えを見る／隠す）</summary>

.. code-block:: cmake
  :caption: TODO 3: CMakeLists.txt
  :name: CMakeLists.txt-target_link_libraries

  target_link_libraries(Tutorial PUBLIC MathFunctions)

.. raw:: html

  </details>

さらにライブラリのヘッダ・ファイルが格納されている場所を教えてあげる必要があります。
:command:`target_include_directories` コマンドを変更して、サブディレクトリの ``MathFunctions``  をインクルード・ディレクトリとして追加し、ヘッダ・ファイルの ``MathFunctions.h`` が見つかるようにします。

.. raw:: html

  <details><summary>TODO 4: （クリックして答えを見る／隠す）</summary>

.. code-block:: cmake
  :caption: TODO 4: CMakeLists.txt
  :name: CMakeLists.txt-target_include_directories-step2

  target_include_directories(Tutorial PUBLIC
                            "${PROJECT_BINARY_DIR}"
                            "${PROJECT_SOURCE_DIR}/MathFunctions"
                            )

.. raw:: html

  </details>

これでライブラリが使えるようになりました。
ソース・ファイルの ``tutorial.cxx`` でヘッダ・ファイルの ``MathFunctions.h`` をインクルードして下さい：

.. raw:: html

  <details><summary>TODO 5: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/tutorial.cxx
  :caption: TODO 5: tutorial.cxx
  :name: CMakeLists.txt-include-MathFunctions.h
  :language: cmake
  :start-after: #include <string>
  :end-before: #include "TutorialConfig.h"

.. raw:: html

  </details>

最後に、``sqrt`` 関数をラッパー関数の ``mathfunctions::sqrt`` で置き換えます。

.. raw:: html

  <details><summary>TODO 6: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/tutorial.cxx
  :caption: TODO 6: tutorial.cxx
  :name: CMakeLists.txt-option
  :language: cmake
  :start-after:   const double inputValue = std::stod(argv[1]);
  :end-before: std::cout

.. raw:: html

  </details>

演習２ - オプションを追加する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now let us add an option in the MathFunctions library to allow developers to select either the custom square root implementation or the built in standard implementation.
While for the tutorial there really isn't any need to do so, for larger projects this is a common occurrence.

CMake can do this using the :command:`option` command. This gives users a variable which they can change when configuring their cmake build.
This setting will be stored in the cache so that the user does not need to set the value each time they run CMake on a build directory.

Goal
----

Add the option to build without ``MathFunctions``.


Helpful Resources
-----------------

* :command:`if`
* :command:`option`
* :command:`target_compile_definitions`

Files to Edit
-------------

* ``MathFunctions/CMakeLists.txt``
* ``MathFunctions/MathFunctions.cxx``

Getting Started
---------------

Start with the resulting files from Exercise 1. Complete ``TODO 7`` through ``TODO 14``.

First create a variable ``USE_MYMATH`` using the :command:`option` command in ``MathFunctions/CMakeLists.txt``.
In that same file, use that option to pass a compile definition to the ``MathFunctions`` library.

Then, update ``MathFunctions.cxx`` to redirect compilation based on ``USE_MYMATH``.

Lastly, prevent ``mysqrt.cxx`` from being compiled when ``USE_MYMATH`` is on by making it its own library inside of the ``USE_MYMATH`` block of ``MathFunctions/CMakeLists.txt``.

Build and Run
-------------

Since we have our build directory already configured from Exercise 1, we can rebuild by simply calling the following:

.. code-block:: console

  cd ../Step2_build
  cmake --build .

Next, run the ``Tutorial`` executable on a few numbers to verify that it's still correct.

Now let's update the value of ``USE_MYMATH`` to ``OFF``.
The easiest way is to use the :manual:`cmake-gui <cmake-gui(1)>` or  :manual:`ccmake <ccmake(1)>` if you're in the terminal.
Or, alternatively, if you want to change the option from the command-line, try:

.. code-block:: console

  cmake ../Step2 -DUSE_MYMATH=OFF

Now, rebuild the code with the following:

.. code-block:: console

  cmake --build .

Then, run the executable again to ensure that it still works with ``USE_MYMATH`` set to ``OFF``.
Which function gives better results, ``sqrt`` or ``mysqrt``?

Solution
--------

The first step is to add an option to ``MathFunctions/CMakeLists.txt``.
This option will be displayed in the :manual:`cmake-gui <cmake-gui(1)>` and :manual:`ccmake <ccmake(1)>` with a default value of ``ON`` that can be changed by the user.

.. raw:: html

  <details><summary>TODO 7: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/MathFunctions/CMakeLists.txt
  :caption: TODO 7: MathFunctions/CMakeLists.txt
  :name: CMakeLists.txt-option-library-level
  :language: cmake
  :start-after: # should we use our own math functions
  :end-before: if (USE_MYMATH)

.. raw:: html

  </details>

Next, make building and linking our library with ``mysqrt`` function conditional using this new option.

Create an :command:`if` statement which checks the value of ``USE_MYMATH``.
Inside the :command:`if` block, put the :command:`target_compile_definitions` command with the compile definition ``USE_MYMATH``.

.. raw:: html

  <details><summary>TODO 8: （クリックして答えを見る／隠す）</summary>

.. code-block:: cmake
  :caption: TODO 8: MathFunctions/CMakeLists.txt
  :name: CMakeLists.txt-USE_MYMATH

  if (USE_MYMATH)
    target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
  endif()

.. raw:: html

  </details>

When ``USE_MYMATH`` is ``ON``, the compile definition ``USE_MYMATH`` will be set.
We can then use this compile definition to enable or disable sections of our source code.

The corresponding changes to the source code are fairly straightforward.
In ``MathFunctions.cxx``, we make ``USE_MYMATH`` control which square root function is used:

.. raw:: html

  <details><summary>TODO 9: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/MathFunctions/MathFunctions.cxx
  :caption: TODO 9: MathFunctions/MathFunctions.cxx
  :name: MathFunctions-USE_MYMATH-if
  :language: c++
  :start-after: which square root function should we use?
  :end-before: }

.. raw:: html

  </details>

Next, we need to include ``mysqrt.h`` if ``USE_MYMATH`` is defined.

.. raw:: html

  <details><summary>TODO 10: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/MathFunctions/MathFunctions.cxx
  :caption: TODO 10: MathFunctions/MathFunctions.cxx
  :name: MathFunctions-USE_MYMATH-if-include
  :language: c++
  :start-after: include <cmath>
  :end-before: namespace mathfunctions

.. raw:: html

  </details>

Finally, we need to include ``cmath`` now that we are using ``std::sqrt``.

.. raw:: html

  <details><summary>TODO 11: （クリックして答えを見る／隠す）</summary>

.. code-block:: c++
  :caption: TODO 11 : MathFunctions/MathFunctions.cxx
  :name: tutorial.cxx-include_cmath

  #include <cmath>

.. raw:: html

  </details>

At this point, if ``USE_MYMATH`` is ``OFF``, ``mysqrt.cxx`` would not be used but it will still be compiled because the ``MathFunctions`` target has ``mysqrt.cxx`` listed under sources.

There are a few ways to fix this.
The first option is to use :command:`target_sources` to add ``mysqrt.cxx`` from within the ``USE_MYMATH`` block.
Another option is to create an additional library within the ``USE_MYMATH`` block which is responsible for compiling ``mysqrt.cxx``.
For the sake of this tutorial, we are going to create an additional library.

First, from within ``USE_MYMATH`` create a library called ``SqrtLibrary`` that has sources ``mysqrt.cxx``.

.. raw:: html

  <details><summary>TODO 12: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/MathFunctions/CMakeLists.txt
  :caption: TODO 12 : MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-add_library-SqrtLibrary
  :language: cmake
  :start-after: # library that just does sqrt
  :end-before: # TODO 7: Link

.. raw:: html

  </details>

Next, we link ``SqrtLibrary`` onto ``MathFunctions`` when ``USE_MYMATH`` is enabled.

.. raw:: html

  <details><summary>TODO 13: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/MathFunctions/CMakeLists.txt
  :caption: TODO 13 : MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-target_link_libraries-SqrtLibrary
  :language: cmake
  :start-after: to tutorial_compiler_flags
  :end-before: endif()

.. raw:: html

  </details>

Finally, we can remove ``mysqrt.cxx`` from our ``MathFunctions`` library source list because it will be pulled in when ``SqrtLibrary`` is included.

.. raw:: html

  <details><summary>TODO 14: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step3/MathFunctions/CMakeLists.txt
  :caption: TODO 14 : MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-remove-mysqrt.cxx-MathFunctions
  :language: cmake
  :end-before: # TODO 1:

.. raw:: html

  </details>

With these changes, the ``mysqrt`` function is now completely optional to whoever is building and using the ``MathFunctions`` library.
Users can toggle ``USE_MYMATH`` to manipulate what library is used in the build.
