ステップ７: システムのイントロスペクションを追加する
====================================================

ターゲットのプラットフォームには無い機能に代わるものをプロジェクトに追加することを考えてみましょう。
たとえば、ターゲットのプラットフォームに ``log`` 関数と ``exp`` 関数があるかどうかを調べて、その結果に応じてコードを追加します。
もちろん、これらの関数はほとんどすべてのプラットフォームで提供されていますが、このチュートリアルでは、それが一般的なことではないことを前提とします。

演習１ - 利用可否を評価する
^^^^^^^^^^^^^^^^^^^^^^^^^^^

目標
----

システムの依存状態に応じて実装を変更する。

参考情報
--------

* :module:`CheckCXXSourceCompiles`
* :command:`target_compile_definitions`

編集するファイル
----------------

* ``MathFunctions/CMakeLists.txt``
* ``MathFunctions/mysqrt.cxx``

始める
------

``Step7`` のディレクトリにあるソース・コードから始めます。
この演習では ``TODO 1`` から ``TODO 5`` まで進めて下さい。

まず、``MathFunctions/CMakeLists.txt`` を編集します。
:module:`CheckCXXSourceCompiles` というモジュールを :command:`include` コマンドでロードします。
それから、``check_cxx_source_compiles`` を使って関数の ``log`` と ``exp`` が ``cmath`` から利用できるかどうかをチェックします。
これらの関数が利用できる場合は :command:`target_compile_definitions` コマンドを使い、コンパイル時の定義フラグとして ``HAVE_LOG`` と ``HAVE_EXP`` を追加します。

``MathFunctions/mysqrt.cxx`` の中で ``cmath`` をインクルードします。
それから ``log`` と ``exp`` の関数が利用できることが判明したら、平方根を計算する際にそれらの関数を使うことにします。

ビルドと実行
------------

``Step7_build`` というディレクトリを新たに作成して下さい。
:manual:`cmake <cmake(1)>` コマンドまたは :manual:`cmake-gui <cmake-gui(1)>` を実行してプロジェクトを構成し、選択したビルド・ツールを使ってプロジェクトをビルドして実行形式の ``Tutoriral`` を実行して下さい。

この一連のコマンドライン操作は次のようになります：

.. code-block:: console

  mkdir Step7_build
  cd Step7_build
  cmake ../Step7
  cmake --build .

``sqrt`` と ``mysqrt`` のどちらの関数が良い結果になりましたか？

解決方法
--------

この演習では :module:`CheckCXXSourceCompiles` モジュールの関数を利用するため、まずそれを ``MathFunctions/CMakeLists.txt`` の中でロードしておく必要があります。

.. raw:: html

  <details><summary>TODO 1: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step8/MathFunctions/CMakeLists.txt
  :caption: TODO 1: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-include-check_cxx_source_compiles
  :language: cmake
  :start-after: # does this system provide the log and exp functions?
  :end-before: check_cxx_source_compiles

.. raw:: html

  </details>

次に、``log`` と ``exp`` 関数が利用できるかどうかを ``check_cxx_compiles_source`` という関数でそれぞれテストします。
この関数は、実際にソース・コードをコンパイルする前に、必要となる依存の有無を確認する簡単なテスト・コードを作成してコンパイルします。
このテスト・コードを実行し、結果として得られる二つの CMake 変数 ``HAVE_LOG`` と ``HAVE_EXP`` は、ソース・コードでそれらの関数を利用できるかどうかを表します。

.. raw:: html

  <details><summary>TODO 2: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step8/MathFunctions/CMakeLists.txt
  :caption: TODO 2: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-check_cxx_source_compiles
  :language: cmake
  :start-after: include(CheckCXXSourceCompiles)
  :end-before: # add compile definitions

.. raw:: html

  </details>

次に、これらの CMake 変数をプロジェクトのソース・コードに渡す必要があります。
このようにして、プロジェクトのソース・コードは必要なリソースが利用できることを知ることができるのです。
``log`` と ``exp`` の両方の関数が利用できるならば :command:`target_compile_definitions` コマンドを使って、``HAVE_LOG`` と ``HAVE_EXP`` を ``PRIVATE`` なコンパイル時の定義フラグとして指定できるようにします。

.. raw:: html

  <details><summary>TODO 3: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step8/MathFunctions/CMakeLists.txt
  :caption: TODO 3: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-target_compile_definitions
  :language: cmake
  :start-after: # add compile definitions
  :end-before: # state

.. raw:: html

  </details>

``log`` と ``exp`` の関数を使っている ``mysqrt.cxx`` を変更して ``cmath`` をインクルードする必要があります。

.. raw:: html

  <details><summary>TODO 4: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step8/MathFunctions/mysqrt.cxx
  :caption: TODO 4: MathFunctions/mysqrt.cxx
  :name: MathFunctions/mysqrt.cxx-include-cmath
  :language: c++
  :start-after: #include "mysqrt.h"
  :end-before: include <iostream>

.. raw:: html

  </details>

``log`` と ``exp`` の関数が利用できる場合は ``mysqrt`` 関数でそれらを使って平方根を計算します。
``MathFunctions/mysqrt.cxx`` にある ``mysqrt`` 関数は次のようになります：

.. raw:: html

  <details><summary>TODO 5: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step8/MathFunctions/mysqrt.cxx
  :caption: TODO 5: MathFunctions/mysqrt.cxx
  :name: MathFunctions/mysqrt.cxx-ifdef
  :language: c++
  :start-after: // if we have both log and exp then use them
  :end-before: return result;

.. raw:: html

  </details>
