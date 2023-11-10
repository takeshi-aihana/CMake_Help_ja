ステップ12: デバッグ版とリリース版のパッケージを作成する
========================================================

**注記:** このチュートリアルで紹介するサンプル・コードは single-config のジェネレータ向けのもので、multi-config 対応のジェネレータ（たとえば Visual Studio）では動作しません。

デフォルトで、CMake はビルド・ディレクトリにデバッグ、リリース、MinSizeRel、または RelWithDebInfo のいずれかのモデルの構成が一つだけ含まれるようになっています。
ただし、CPack を設定して複数のビルド・ディレクトリをまとめ、同じプロジェクトで複数のモデルの構成を含む一個のパッケージを作成することは可能です。

その場合は、まずデバッグ・ビルドとリリース・ビルドごとに別のライブラリ名でインストールされるようにする必要があります。
ここでは、たとえばデバッグ版のライブラリには接尾辞として `d` を使うことにしましょう。

プロジェクト最上位にある ``CMakeLists.txt`` の先頭近くで、変数の :variable:`CMAKE_DEBUG_POSTFIX` に接尾辞の `d` をセットします：

.. literalinclude:: Complete/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-CMAKE_DEBUG_POSTFIX-variable
  :language: cmake
  :start-after: project(Tutorial VERSION 1.0)
  :end-before: target_compile_features(tutorial_compiler_flags

さらにターゲットの ``Tutorial`` にもプロパティ :prop_tgt:`DEBUG_POSTFIX` をセットします：

.. literalinclude:: Complete/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-DEBUG_POSTFIX-property
  :language: cmake
  :start-after: # add the executable
  :end-before: # add the binary tree to the search path for include files

さらに ``MathFunctions`` ライブラリにバージョン番号も追加しておきましょう。
``MathFunctions/CMakeLists.txt`` の中で :prop_tgt:`VERSION` と :prop_tgt:`SOVERSION` のプロパティをセットします：

.. literalinclude:: Complete/MathFunctions/CMakeLists.txt
  :caption: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-VERSION-properties
  :language: cmake
  :start-after: # setup the version numbering
  :end-before: # install libs

そして ``Step12`` ディレクトリの下に ``debug`` と ``release`` というサブディレクトリを作成します。
この時点のディレクトリのレイアウトは次のようになります：

.. code-block:: none

  - Step12
     - debug
     - release

ここでデバッグ版とリリース版のビルドを設定します。
変数の :variable:`CMAKE_BUILD_TYPE` を使って構成モデルをセットします：

.. code-block:: console

  cd debug
  cmake -DCMAKE_BUILD_TYPE=Debug ..
  cmake --build .
  cd ../release
  cmake -DCMAKE_BUILD_TYPE=Release ..
  cmake --build .

これでデバッグ版のビルドとリリース版のビルドが完了したので、カスタムの構成ファイルを使って双方のビルドを一個のリリース用パッケージにまとめることができます。
``Step12`` ディレクトリの下に ``MultiCPackConfig.cmake`` というファイルを作成します。
このファイルには、まず :manual:`cmake  <cmake(1)>` コマンドで生成したデフォルトの構成ファイルを含めるようにします。

次に ``CPACK_INSTALL_CMAKE_PROJECTS`` という変数を使って、どちらの構成モデルをインストールするかを指定します。
ここではデバッグ版とリリース版の両方をインストールしたいとします。

.. literalinclude:: Complete/MultiCPackConfig.cmake
  :caption: MultiCPackConfig.cmake
  :name: MultiCPackConfig.cmake
  :language: cmake

``Step12`` ディレクトリで、用意したカスタムの構成ファイルを :option:`--config <cpack --config>`  オプションの引数に指定して :manual:`cpack <cpack(1)>` コマンドを実行して下さい：

.. code-block:: console

  cpack --config MultiCPackConfig.cmake
