ステップ３: ライブラリの利用要件を追加する
==========================================

演習１ - ライブラリの利用要件を追加する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ターゲット・プロパティの「:ref:`利用要件 <Target Usage Requirements>`」（*Usage Requirements*） [#hint_for_usage_requirements]_ を使うと、ライブラリや実行形式のリンクと #include 行をより適切に制御できると共に、「ターゲットが変化する」プロパティを CMake 内でより細かく制御できるようになります。
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

ライブラリに「利用要件」を追加する。

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
ライブラリに独自の「利用要件」を定義させて、必要に応じて別のターゲットにプロパティを渡せるようにします。
この時、``MathFunctions`` ライブラリにはビルドに必要なインクルード・ディレクトリを指定します。
次に、ターゲットである実行形式の ``Tutorial`` は ``MathFunctions`` ライブラリにリンクするだけで、追加のインクルード・ディレクトリについては特に気にする必要はありません。

出発点は ``Step3`` ディレクトリにあるソース・ファイルです。
この演習では ``TODO 1`` から始めて ``TODO 3`` まで進めて下さい。

まず ``MathFunctions/CMakeLists`` に :command:`target_include_directories` コマンドを追加します。
変数の :variable:`CMAKE_CURRENT_SOURCE_DIR` は、現在作業しているソース・ファイルがあるパス名です。

それからプロジェクト最上位の ``CMakeLists.txt`` にある :command:`target_include_directories` コマンドの呼び出しを修正して短くします。

ビルドと実行
-------------

あらたに ``Step3_build`` ディレクトリを作成し、:manual:`cmake <cmake(1)>` コマンドまたは :manual:`cmake-gui <cmake-gui(1)>` を実行してプロジェクトを構成し、選択したビルド・ツールを使うか、またはビルド・ディレクトリから :option:`cmake --build . <cmake --build>` を実行してプロジェクトをビルドします。
この一連のコマンドライン操作は次のようになります：

.. code-block:: console

  mkdir Step3_build
  cd Step3_build
  cmake ../Step3
  cmake --build .

次に、新しくビルドした実行形式の ``Tutorial`` が期待通り動作するか確認して下さい。

解決方法
--------

前のステップのコードを修正して、CMake の最新のアプローチである「利用要件」を使ってみることにしましょう。

まず、``MathFunctions`` ライブラリを利用する（リンクする）場合は :variable:`CMAKE_CURRENT_SOURCE_DIR` を #include する必要がありますが、``Mathfunctions`` ライブラリそのものをビルドする際は必要ないことについて説明しておきたいと思います。
これは、``INTERFACE`` という利用要件で表現します。
``INTERFACE`` はライブラリを利用するユーザに必要なものですが、ライブラリをビルドしてユーザに提供する開発者には必要ないということを覚えておいて下さい。

``MathFunctions/CMakeLists.txt`` の最後で、次のように ``INTERFACE`` キーワードを指定して :command:`target_include_directories` コマンドを呼び出します：

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

ここでは、``MathFunctions`` ライブラリの利用要件を指定したので、プロジェクト最上位の ``CMakeLists.txt`` から  ``EXTRA_INCLUDES`` 変数を安全に削除できます。

この行を削除して下さい：

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

そして ``target_include_directories`` から ``EXTRA_INCLUDES`` 変数のエントリを削除します：

.. raw:: html

  <details><summary>TODO 3: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step4/CMakeLists.txt
  :caption: TODO 3: CMakeLists.txt
  :name: CMakeLists.txt-target_include_directories-remove-EXTRA_INCLUDES
  :language: cmake
  :start-after: # so that we will find TutorialConfig.h

.. raw:: html

  </details>

この手法だと、ターゲットである実行形式がライブラリを利用するために行うことは、ライブラリのターゲット名を指定して :command:`target_link_libraries` コマンドを呼び出すだけです。
もっと大規模なプロジェクトだと、ライブラリの依存関係を手動で指定する従来の方法だと、すぐに複雑化して混乱を引き起こすことになるでしょう。

演習２ - インタフェース・ライブラリに C++ 標準を適用する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

ここでコードを新しいアプローチに切り替えたので、プロパティを複数のターゲットにセットする最新の手法について確認してみましょう。

ライブラリの ``INTERFACE`` を使用するように既存のコードをリファクタリングしましょう。
次のステップで、このライブラリを使って「:manual:`ジェネレータ式 <cmake-generator-expressions(7)>`」の一般的な使用方法を示します。

目標
----

ターゲットに ``INTERFACE`` 型のライブラリを追加して、必要な C++ 標準を指定する。

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

この演習では、``INTERFACE`` 型のライブラリを使って C++11 標準を要求するようにコードをリファクタリングします。

演習１の ``STEP3`` の最後に残っていた部分から始めます。
そして ``TODO 4`` から ``TODO 7`` まで完了して下さい。

まずプロジェクト最上位の ``CMakeLists.txt`` ファイルを編集します。
``tutorial_compiler_flags`` という ``INTERFACE`` 型のライブラリをターゲットとしてビルドし、ターゲットのコンパイラ機能として ``cxx_std_11`` を指定します。

``CMakeLists.txt`` と ``MathFunctions/CMakeLists.txt`` を変更して、全てのターゲットで ``tutorial_compiler_flags`` を引数に :command:`target_link_libraries`  コマンドを呼び出すようにします。

ビルドと実行
------------

演習１で既にビルド・ディレクトリを構成しているので、次のコマンドラインを実行してコードを再ビルドするだけです：

.. code-block:: console

  cd Step3_build
  cmake --build .

次に、新しくビルドした実行形式の ``Tutorial`` が期待通り動作するか確認して下さい。

解決方法
--------

演習１からのコードを、INTERFACE 型のライブラリを使って C++11 標準を要求するように更新しましょう。

まず、変数の :variable:`CMAKE_CXX_STANDARD` と :variable:`CMAKE_CXX_STANDARD_REQUIRED` に対する :command:`set` コマンド二つの呼び出しを削除する必要があります。
削除の対象となる行は次のとおりです：

.. literalinclude:: Step3/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-CXX_STANDARD-variable-remove
  :language: cmake
  :start-after: # specify the C++ standard
  :end-before: # configure a header file

さらに ``tutorial_compiler_flags`` という INTERFACE 型のライブラリを作成する必要があります。
そして :command:`target_compile_features` コマンドを使って、コンパイラ・フラグである ``cxx_std_11`` を指定します。


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

最後に INTERFACE 型のライブラリを設定したら、実行形式の ``Tutorial``、``SqrtLibrary`` ライブラリ、そして ``MathFunctions`` ライブラリをそれぞれ新しい ``tutorial_compiler_flags`` ライブラリとリンクして下さい。
それぞれ次のようになります：

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

さらに：

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

最後に：

.. raw:: html

  <details><summary>TODO 7: （クリックして答えを見る／隠す）</summary>

.. literalinclude:: Step4/MathFunctions/CMakeLists.txt
  :caption: TODO 7: MathFunctions/CMakeLists.txt
  :name: MathFunctions-SqrtLibrary-target_link_libraries-step4
  :language: cmake
  :start-after: # link MathFunctions to tutorial_compiler_flags

.. raw:: html

  </details>

以上の変更で、すべてのコードのビルドで C++11 の標準が要求されます。
この手法であれば、どのターゲットが C++11 の標準に準拠しているかを具体的に指定できるという点に留意しておいて下さい。
さらに、この INTERFASE 型のライブラリの中に唯一のソースコードを生成します [#comment_for_translation_01]_ 。


.. rubric:: 日本語訳注記

.. [#hint_for_usage_requirements] `CMake再入門メモ <https://zenn.dev/rjkuro/articles/054dab5b0e4f40#build-specification%E3%81%A8usage-requirement>`_ 参照。
.. [#comment_for_translation_01] 意味不明。原文は "In addition, we create a single source of truth in our interface library."。
