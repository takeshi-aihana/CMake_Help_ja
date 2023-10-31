ステップ５: インストールとテスト
================================

演習１ - インストールのルール
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

多くの場合、実行形式をビルドするだけではなく、それをインストールしなければならないケースがあります。
CMake では :command:`install` コマンドでインストールのルールを指定できます。
ビルドしたものをローカルにインストールする処理を CMake に追加することは、インストール先とインストールするターゲットやファイルを指定することと同じくらい簡単です。

目標
----

ビルドした実行形式の ``Tutorial`` とライブラリの ``MathFunctions`` をインストールする。

参考情報
--------

* :command:`install`

編集するファイル
----------------

* ``MathFunctions/CMakeLists.txt``
* ``CMakeLists.txt``

始める
------

``Step5`` のディレクトリにあるソース・コードから始めます。
この演習では ``TODO 1`` から ``TODO 4`` まで進めて下さい。

まず、``MathFunctions/CMakeLists.txt`` を更新して二つのライブラリ ``MathFunctions`` と ``tutorial_compiler_flags`` を ``lib`` というディレクトリにインストールします。
同様にヘッダ・ファイルの ``MathFunctions.h`` を ``include`` というディレクトリにインストールするためのルールを追加します。

次にプロジェクト最上位の ``CMakeLists.txt`` を更新して実行形式の ``Tutorial`` を ``bin`` というディレクトリにインストールします。
最後に、全てのヘッダ・ファイルを ``include`` というディレクトリにインストールして下さい。
ここで ``TutorialConfig.h`` というヘッダ・ファイルは :variable:`PROJECT_BINARY_DIR` にあることに注意して下さい。

ビルドと実行
------------

``Step5_build`` というディレクトリを新たに作成して下さい。
:manual:`cmake <cmake(1)>` コマンドまたは :manual:`cmake-gui <cmake-gui(1)>` を実行してプロジェクトを構成し、選択したビルド・ツールを使ってプロジェクトをビルドします。

それから :manual:`cmake  <cmake(1)>` コマンドに :option:`--install <cmake --install>` オプションを付けてインストールを実行します
（CMake バージョン 3.15 から。これより古いバージョンの CMake では ``make install`` を実行する必要あり）。
このステップは適切なヘッダ・ファイルとライブラリ、そして実行形式をインストールします。
たとえば：

.. code-block:: console

  cmake --install .

複数ある configuration ツールを使用している場合（たとえば Visual Studio など）は、:option:`--config <cmake--build --config>` オプションを使って使用する configuration を指定することを忘れないで下さい。

.. code-block:: console

  cmake --install . --config Release

IDE を使っている場合は ``INSTALL`` ターゲットをビルドするだけです。
次のようにコマンドラインからも同じ ``install`` ターゲットをビルドできます：

.. code-block:: console

  cmake --build . --target install --config Debug

CMake の変数 :variable:`CMAKE_INSTALL_PREFIX` はファイルをインストール先の Prefix を決定するために使用します。
:option:`cmake --install` を実行する場合、この変数の値は :option:`--prefix <cmake--install --prefix>`  オプションで上書きできます。
たとえば：

.. code-block:: console

  cmake --install . --prefix "/home/myuser/installdir"

インストール先のディレクトリへ移動して、インストールされた ``Tutorial`` が実行できることを確認して下さい。

解決方法
--------

このプロジェクトのインストール・ルールは非常に簡単なものです：

* ライブラリの ``MathFunctions`` は、二つのライブラリ・ファイルとヘッダ・ファイルをそれぞれ ``lib`` と ``include`` ディレクトリにインストールする

* 実行形式の ``Tutorial`` は、実行形式のファイルと設定済みのヘッダ・ファイルをそれぞれ ``bin`` と ``include`` ディレクトリにインストールする

そのため ``MathFunctions/CMakeLists.txt`` の最後に以下を追加します：

.. raw:: html

  <details><summary>TODO 1: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step6/MathFunctions/CMakeLists.txt
  :caption: TODO 1: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-install-TARGETS
  :language: cmake
  :start-after: # install libs
  :end-before: # install include headers

.. raw:: html

  </details>

そして：

.. raw:: html

  <details><summary>TODO 2: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step6/MathFunctions/CMakeLists.txt
  :caption: TODO 2: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-install-headers
  :language: cmake
  :start-after: # install include headers

.. raw:: html

  </details>

実行形式 ``Tutorial`` と設定済みのヘッダ・ファイルをインストールするルールはにています。
このプロジェクト最上位の ``CMakeLists.txt`` の最後に以下を追加します：

.. raw:: html

  <details><summary>TODO 3,4: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step6/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: TODO 3,4: CMakeLists.txt-install-TARGETS
  :language: cmake
  :start-after: # add the install targets
  :end-before: # TODO 1: Replace enable_testing() with include(CTest)

.. raw:: html

  </details>

基本的なローカル・インストールを行うために必要なことはこれだけです。

.. _`Tutorial Testing Support`:

演習２ - テストのサポート
^^^^^^^^^^^^^^^^^^^^^^^^^

CTest はいろいろなテストをプロジェクトで簡単に管理する方法を提供しています。
テストは :command:`add_test` コマンドで追加することができます。
このチュートリアルでは詳しく説明していませんが、CTest と :module:`GoogleTest` のようなその他のテスト用フレームワークとの間には多くの互換性があります。

目標
----

CTest を使ってプロジェクトの実行形式の Unit テストを作成する。

参考情報
--------

* :command:`enable_testing`
* :command:`add_test`
* :command:`function`
* :command:`set_tests_properties`
* :manual:`ctest <ctest(1)>`

編集するファイル
----------------

* ``CMakeLists.txt``

始める
------

``Step5`` のディレクトリにあるソース・コードから始めます。
この演習では ``TODO 5`` から ``TODO 9`` まで進めて下さい。

まず、テストを有効にうする必要があります。
それから :command:`add_test` コマンドを使ってプロジェクトにテストを追加していきます。
このプロジェクトには3つの簡単なテストを追加します。
必要であればご自身でテストを追加することも可能です。

ビルドと実行
------------

ビルド・ディレクトリに移動して、プロジェクトをリビルドします。
それから :manual:`ctest(1)` コマンドを次のようにして実行します： :option:`ctest -N` と :option:`ctest -VV` 。
複数ある configuration ツールを使用している場合（たとえば Visual Studio など）は、:option:`-C \<mode\> <ctest -C>` オプションを使ってテストする構成を指定して下さい。
たとえばデバッグ・モードでテストを実行する場合は、（デバッグ用のサブディレクトリではなく）ビルド・ディレクトリから ``ctest -C Debug -VV`` を実行します。
対してリリース・モードも同じビルド・ディレクトリから実行しますが、``-C Release`` として実行します。
あるいは IDE の場合は ``RUN_TESTS`` というターゲットをビルドして下さい。

解決方法
--------

このプロジェクトをテストしましょう。
まずプロジェクト最上位の ``CMakeLists.txt`` の最後で、:command:`enable_testing` を呼び出してテストを有効にします。

.. raw:: html

  <details><summary>TODO 5: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step6/CMakeLists.txt
  :caption: TODO 5: CMakeLists.txt
  :name: CMakeLists.txt-enable_testing
  :language: cmake
  :start-after: # enable testing
  :end-before: # does the application run

.. raw:: html

  </details>

テストを有効にしたら、プロジェクトのアプリケーションが正しく動作していることを確認するために、基本的なテストをいくつか追加します。
まず最初に、実行形式の ``Tutorial`` に引数として 25 を渡して実行するテストを :command:`add_test` コマンドで追加します。
このテストでは、``Tutorial`` の実際の計算結果はチェックしません。
このテストではアプリケーションが実行され、SIGSEGV などでクラッシュせずに、返り値が 0 であることを確認します。
これが CTest によるテストの基本形です。

.. raw:: html

  <details><summary>TODO 6: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step6/CMakeLists.txt
  :caption: TODO 6: CMakeLists.txt
  :name: CMakeLists.txt-test-runs
  :language: cmake
  :start-after: # does the application run
  :end-before: # does the usage message work

.. raw:: html

  </details>

次に :prop_test:`PASS_REGULAR_EXPRESSION` というテスト用プロパティを使って、テストの結果に特定の文字列が含まれていることを確認してみましょう。
この場合、間違った引数を渡して実行した時に用法（*Usage*）のメッセージが出力されることを確認します。

.. raw:: html

  <details><summary>TODO 7: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step6/CMakeLists.txt
  :caption: TODO 7: CMakeLists.txt
  :name: CMakeLists.txt-test-usage
  :language: cmake
  :start-after: # does the usage message work?
  :end-before: # define a function to simplify adding tests

.. raw:: html

  </details>

次に追加するテストは、計算結果が正しい平方根であるかを確認します。

.. raw:: html

  <details><summary>TODO 8: （クリックして答えを見る／隠す）</summary>

.. code-block:: cmake
  :caption: TODO 8: CMakeLists.txt
  :name: CMakeLists.txt-test-standard

  add_test(NAME StandardUse COMMAND Tutorial 4)
  set_tests_properties(StandardUse
    PROPERTIES PASS_REGULAR_EXPRESSION "4 is 2"
    )

.. raw:: html

  </details>

この1個のテストだけで、あらゆる引数に対して正しく計算しているかを判定するには不十分です。
これを検証するために、もっとテストを追加する必要があります。
複数のテストを簡単に追加するに、アプリケーションを実行して、引数に対して計算された平方根の値が正しいかどうかを顕彰する ``do_test`` という関数を作成します。
この ``do_test`` を呼び出すたびに、関数に渡した引数からテスト名、テストに渡す入力パラメータ、そして期待する結果を持つテストが自動的にプロジェクトに追加されます。

.. raw:: html

  <details><summary>TODO 9: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step6/CMakeLists.txt
  :caption: TODO 9: CMakeLists.txt
  :name: CMakeLists.txt-generalized-tests
  :language: cmake
  :start-after: # define a function to simplify adding tests

.. raw:: html

  </details>
