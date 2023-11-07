ステップ９: インストーラのパッケージ化
======================================

次に、このプロジェクトを他の人たちに配布して使ってもらえるようにしたいとします。
いろいろなプラットフォーム向けにバイナリとソースの両方の配布物（パッケージ）を提供することにしましょう。
これは、以前のチュートリアル「:guide:`tutorial/Installing and Testing`」で実施した、ソース・ファイルからビルドしたバイナリ・ファイルをインストールして配布する場合とは少し意味合いが異なります。
ここでは、バイナリ・ファイルのインストールとパッケージ管理機能をサポートするインストール専用パッケージをビルドできるようにします。
これを実現するために、CPack を使ってプラットフォーム別のインストーラを作成します。
具体的には、プロジェクト最上位にある ``CMakeLists.txt`` ファイルの最後に数行のコマンドを追加して下さい：

.. literalinclude:: Step10/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-include-CPack
  :language: cmake
  :start-after: # setup installer

追加するコマンドは、これだけです。
:module:`InstallRequiredSystemLibraries` というモジュールをロードするところから始まります。
このモジュールには、現在のプラットフォーム向けのプロジェクトで必要なラインタイム・ライブラリが含まれています。
次に、このプロジェクトのライセンスとバージョン情報を格納する CPack の変数をいくつかセットします。
バージョン情報は、前半のチュートリアルで既にセットしています。このステップではプロジェクト最上位のソース・ディレクトリに ``License.txt`` というファイルが含まれていることを確認して下さい。
CPack 変数の :variable:`CPACK_SOURCE_GENERATOR` でソース・パッケージのファイル形式を指定します。

最後に、これら CPack の変数と現在のシステム向けのその他のプロパティを使ってインストーラをセットアップするために :module:`CPack module <CPack>` というモジュールをロードします。

次のステップでは、通常の方法でプロジェクトをビルドしてから :manual:`cpack <cpack(1)>` コマンドを実行します。
バイナリ・パッケージをビルドする際は、ビルド・ディレクトリで次のコマンドを実行します：

.. code-block:: console

  cpack

この時、ジェネレータを指定する場合は :option:`-G <cpack -G>` オプションを使います。
multi-config に対応したビルドの場合は、:option:`-C <cpack -C>` オプションを使って構成を指定します。
たとえば：

.. code-block:: console

  cpack -G ZIP -C Debug

ここで利用可能なジェネレータについては :manual:`cpack-generators(7)` を参照するか、または :option:`cpack --help` を実行して確認してみて下さい。
ZIP などの :cpack_gen:`archive generator <CPack Archive Generator>` は「インストールされる」全てのファイルの圧縮アーカイブを作成します。

「すべての」ソース・ファイルのアーカイブを作成する場合は、次のように実行します：

.. code-block:: console

  cpack --config CPackSourceConfig.cmake

あるいは ``make package`` コマンドを実行するか、または IDE から ``Package`` ターゲットを右クリックして ``Build Project`` を選択します。

ビルド・ディレクトリにあるインストーラを実行してみて下さい。
そしてインストールされた実行形式を起動して、動作することを確認してみて下さい。
