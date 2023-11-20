.. cmake-manual-description: CMake GUI コマンドライン・リファレンス

cmake-gui(1)
************

概要
====

.. parsed-literal::

 cmake-gui [<options>]
 cmake-gui [<options>] -B <path-to-build> [-S <path-to-source>]
 cmake-gui [<options>] <path-to-source | path-to-existing-build>
 cmake-gui [<options>] --browse-manual [<filename>]

説明
====

:program:`cmake-gui` は CMake のグラフィカル・ユーザ・インタフェース（GUI）です。
プロジェクトの構成に含まれる各種設定を対話的に指定できます。
このインタフェースを実行しているときは、ウィンドウの下部に簡単な説明が表示されます。

CMake はクロス・プラットフォーム対応のビルドシステムのジェネレータ（Generator）です。
プロジェクトは、ソースツリーの各ディレクトリにある ``CMakeLists.txt`` という名前の、プラットフォームに依存しないリストファイルを使用してビルド方法や手順を指定します。
ユーザは CMake でプロジェクトをビルドして、実行するプラットフォームに対応したビルドシステムを生成します。

オプション
==========

.. program:: cmake-gui

.. option:: -S <path-to-source>

 CMake プロジェクトの root ディレクトリ（ソースツリー）を指定する。

.. option:: -B <path-to-build>

 CMake がビルドツリーとして使用するディレクトリを指定する。

 もしディレクトリがまだ存在していなかったら、CMake が作成する。

.. option:: --preset=<preset-name>

 使用する :manual:`presets <cmake-presets(7)>` の名前を指定する（プロジェクトにプリセットファイルが存在する場合）。

.. option:: --browse-manual [<filename>]

 ブラウザで CMake のリファレンス・マニュアルを開いて終了する。
 もし ``<filename>`` が指定された場合は、そのファイルを開く。

.. include:: OPTIONS_HELP.txt

関連項目
========

.. include:: LINKS.txt
