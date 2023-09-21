.. title:: CMake リファレンス・ドキュメント

はじめに
############

CMake はソースコードのビルドを制御・管理するツールです。はじめ CMake は ``Makefile`` に似た文法の制御コードを生成するツールとして設計されましたが、現在の CMake は ``Ninja`` といった「現代的なビルドシステム」を生成する他、Visual Studio や Xcode のような IDE で使用するプロジェクトファイルを生成できるようになっています。

CMake は C と C++ 言語のビルドで広く利用されていますが、他のプログラミング言語のビルドでも利用できる場合があります。

はじめて CMake を利用しょうとする人たちは、さまざまな目標をお持ちかと思います。たとえばインターネットからダウンロードしたソースコードをビルドする方法を知りたいのであれば、
:guide:`ユーザ操作ガイド` から始めてみて下さい。これは :manual:`cmake(1)` や :manual:`cmake-gui(1)` というコマンドを使う際に必要な手順、あるいはどのジェネレータを選択すればよいか、そしてソースコードをビルドする方法について詳しく説明しています。

:guide:`依存関係を利用するガイド` ではサードパーティ製のライブラリを使用するような開発者を対象としています。

CMake を使うプロジェクトを始めようとする開発者であれば :guide:`CMake チュートリアル` が出発点として最適です。マニュアルの :manual:`cmake-buildsystem(7)` はビルドシステムis aimed at developers expanding their knowledge of maintaining a buildsystem and becoming familiar with the build targets that can be represented in CMake.
The :manual:`cmake-packages(7)` manual explains how to create packages which can easily be consumed by third-party CMake-based buildsystems.


For developers starting a project using CMake, the :guide:`CMake Tutorial`
is a suitable starting point.  The :manual:`cmake-buildsystem(7)`
manual is aimed at developers expanding their knowledge of maintaining
a buildsystem and becoming familiar with the build targets that
can be represented in CMake.  The :manual:`cmake-packages(7)` manual
explains how to create packages which can easily be consumed by
third-party CMake-based buildsystems.

Command-Line Tools
##################

.. toctree::
   :maxdepth: 1

   /manual/cmake.1
   /manual/ctest.1
   /manual/cpack.1

Interactive Dialogs
###################

.. toctree::
   :maxdepth: 1

   /manual/cmake-gui.1
   /manual/ccmake.1

Reference Manuals
#################

.. toctree::
   :maxdepth: 1

   /manual/cmake-buildsystem.7
   /manual/cmake-commands.7
   /manual/cmake-compile-features.7
   /manual/cmake-configure-log.7
   /manual/cmake-developer.7
   /manual/cmake-env-variables.7
   /manual/cmake-file-api.7
   /manual/cmake-generator-expressions.7
   /manual/cmake-generators.7
   /manual/cmake-language.7
   /manual/cmake-modules.7
   /manual/cmake-packages.7
   /manual/cmake-policies.7
   /manual/cmake-presets.7
   /manual/cmake-properties.7
   /manual/cmake-qt.7
   /manual/cmake-server.7
   /manual/cmake-toolchains.7
   /manual/cmake-variables.7
   /manual/cpack-generators.7

.. only:: not man

 Guides
 ######

 .. toctree::
    :maxdepth: 1

    /guide/tutorial/index
    /guide/user-interaction/index
    /guide/using-dependencies/index
    /guide/importing-exporting/index
    /guide/ide-integration/index

.. only:: html or text

 Release Notes
 #############

 .. toctree::
    :maxdepth: 1

    /release/index

.. only:: html

 Index and Search
 ################

 * :ref:`genindex`
 * :ref:`search`
