ステップ８: カスタム・コマンドと一時ファイルを追加する
======================================================

このチュートリアルでは、プラットフォームで提供している ``log`` や ``exp`` 関数を使わず、代わりに ``mysqrt`` 関数で使用する計算値テーブルをビルドする時に生成しておくものとします。
このステップでは、ビルドの過程でテーブルを生成し、そのテーブルをチュートリアルのアプリケーションの中に取り込んでコンパイルします。

まず、``MathFunctions/CMakeLists.txt`` の中にある ``log`` と ``exp`` の関数の利用可否をチェックするコマンド部分を削除しましょう。
それからソース・ファイルの ``mysqrt.cxx`` から ``HAVE_LOG`` と ``HAVE_EXP`` のチェックも削除します。
同時に :code:`#include <cmath>` のインクルードも削除できます。

``MathFunctions`` サブディレクトリには、計算値のテーブルを生成するために新しく ``MakeTable.cxx`` というファイルが提供されています。

このファイルを見てみると、計算値のテーブルを生成する C++ の関数があり、引数として渡されたファイルに出力しているのがわかります。

次のステップで ``MathFunctions/MakeTable.cmake`` を作成します。
そして ``MakeTable`` という実行形式をビルドするための CMake コマンドを追加して、ビルドの過程でそれを実行するようにします。
これを実現するには、いくつかのコマンドが必要です。

まず ``MakeTable`` という実行形式の作成を CMake に指示します。

.. literalinclude:: Step9/MathFunctions/MakeTable.cmake
  :caption: MathFunctions/MakeTable.cmake
  :name: MathFunctions/MakeTable.cmake-add_executable-MakeTable
  :language: cmake
  :start-after: # first we add the executable that generates the table
  :end-before: target_link_libraries

実行形式をビルドしたら、:command:`target_link_libraries` コマンドを使って実行形式に ``tutorial_compiler_flags`` ライブラリをリンクします。

.. literalinclude:: Step9/MathFunctions/MakeTable.cmake
  :caption: MathFunctions/MakeTable.cmake
  :name: MathFunctions/MakeTable.cmake-link-tutorial-compiler-flags
  :language: cmake
  :start-after: add_executable
  :end-before: # add the command to generate

そして、``MakeTable`` を実行して ``Table.h`` というファイルを作成する方法を指定する CMake のカスタム・コマンドを追加します。

.. literalinclude:: Step9/MathFunctions/MakeTable.cmake
  :caption: MathFunctions/MakeTable.cmake
  :name: MathFunctions/MakeTable.cmake-add_custom_command-Table.h
  :language: cmake
  :start-after: # add the command to generate the source code

なお、ソース・ファイルの ``mysqrt.cxx`` が、ここで作成した ``Table.h`` に依存していることを CMake に知らせる必要があります。
これは ``Sqrtlibrary`` ライブラリに対するソースのリストに ``Table.h`` を追加することで実現できます。

.. literalinclude:: Step9/MathFunctions/CMakeLists.txt
  :caption: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-add_library-Table.h
  :language: cmake
  :start-after:   # library that just does sqrt
  :end-before: # state that we depend on

また、インクルード・ディレクトリのリストに、プロジェクトのビルド・ディレクトリを追加する必要があります。
これにより、ビルド時に ``Table.h`` を見つけることができ、``mysqrt.cxx`` からインクルードできるようになります。

.. literalinclude:: Step9/MathFunctions/CMakeLists.txt
  :caption: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-target_include_directories-Table.h
  :language: cmake
  :start-after: # state that we depend on our bin
  :end-before: target_link_libraries

最後のステップとして、``MathFunctions/CMakeLists.txt`` の先頭で ``MakeTable.cmake`` をインクルードして下さい。

.. literalinclude:: Step9/MathFunctions/CMakeLists.txt
  :caption: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-include-MakeTable.cmake
  :language: cmake
  :start-after: # generate Table.h
  :end-before: # library that just does sqrt

これで生成したテーブルを利用する準備が整ったので、実際に使ってみましょう。
まず ``mysqrt.cxx`` を修正して ``Table.h`` をインクルードして下さい。
次にテーブルを利用するように ``mysqrt`` 関数を書き換えて下さい：

.. literalinclude:: Step9/MathFunctions/mysqrt.cxx
  :caption: MathFunctions/mysqrt.cxx
  :name: MathFunctions/mysqrt.cxx
  :language: c++
  :start-after: // a hack square root calculation using simple operations

:manual:`cmake <cmake(1)>` コマンドまたは :manual:`cmake-gui <cmake-gui(1)>` を実行してプロジェクトを構成し、選択したビルド・ツールを使ってプロジェクトをビルドして下さい。

このプロジェクトのビルドが始まると、まず実行形式の ``MakeTable`` を作成します。
それから ``MakeTable`` を実行して ``Table.h`` を作成します。
最後に、``Table.h`` をインクルードした ``mysqrt.cxx`` をコンパイルして、``MathFunctions`` ライブラリを作成します。
実行形式の ``Tutorial`` を実行して、テーブルが使用されていることを確認してみて下さい。
