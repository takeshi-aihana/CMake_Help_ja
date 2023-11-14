.. cmake-manual-description: CMake（Curses Dialog）コマンド・リファレンス

ccmake(1)
*********

概要
====

.. parsed-literal::

 ccmake [<options>] -B <path-to-build> [-S <path-to-source>]
 ccmake [<options>] <path-to-source | path-to-existing-build>

説明
====

:program:`ccmake` は CMake の Curses 対応のユーザ・インタフェースです。
プロジェクトの構成に含まれる各種設定を Curses を使って対話的に指定できます
このインタフェース実行しているときは、端末の下部に簡単な説明が表示されます

CMake はクロス・プラットフォーム対応のビルドシステムのジェネレータ（Generator）です。
プロジェクトは、ソースツリーの各ディレクトリにある ``CMakeLists.txt`` という名前の、プラットフォームに依存しないリストファイルを使用してビルド方法や手順を指定します。
ユーザは CMake でプロジェクトをビルドして、実行するプラットフォームに対応したビルドシステムを生成します。

オプション
==========

.. program:: ccmake

.. include:: OPTIONS_BUILD.txt

.. include:: OPTIONS_HELP.txt

関連項目
========

.. include:: LINKS.txt
