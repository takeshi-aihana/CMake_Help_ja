ステップ１: 基本を学ぶ出発点
============================

CMake の基本を学ぶために、どこから始めればよいでしょう？
このステップでは、CMake の基本的な構文、CMake のコマンド、そして CMake の変数についてその一部を紹介します。
これらの概念について紹介しながら、3つの演習にとりくみ、簡単な CMake プロジェクトを作成してみます。

このステップにある演習はそれぞれ、その背景にあるもの（前提や条件）について説明するところから始まります。
そのあとで演習の目標と、参考になるいろいろな情報について説明します。
「``編集するファイル``」のセクションで説明しているファイルは ``Step1`` ディレクトリの中にあり、それらには ``TODO`` 付きのコメントが複数あります。
``TODO`` 付きのコメントはそれぞれ、変更や修正が必要な行があることを意味しています。
``TODO`` 付きのコメントについては、まず ``TODO 1`` を完了させてから ``TODO 2`` に進むように、番号順で完了させることが目標です。
「``始める``」のセクションでは、演習を通じて有益なヒントを案内します。
そして「``ビルドと実行``」のセクションでは演習をビルドしてテストする方法について、順を追って説明します。
最後に、各演習の終わりでその解決方法について説明します。

また、このチュートリアルのステップは、それぞれ次のステップにつながっていることに注意して下さい。
したがって、例えば ``Step2`` で演習するコードは ``Step1`` の演習を解決した内容から始まります。

演習１ - 簡単なプロジェクトをビルドする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

最も簡単で基本的な CMake プロジェクトは、一個のソース・ファイルから一個の実行形式をビルドするというものです。
このように単純なプロジェクトの場合、ビルドに必要なものは３個のコマンドを記述した ``CMakeLists.txt`` ファイルだけです。

**注意:** CMake でサポートしているのは大文字、小文字、そして大小文字が混ざったコマンドですが、小文字のコマンドが他よりも優先され、本チュートリアルでもこれに従います。

すべてのプロジェクトでは、``CMakeLists.txt`` の先頭で必ず :command:`cmake_minimum_required` コマンドを使って CMake の最小バージョンを指定して下さい。
これによりプロジェクトのポリシーが確立し、それ以降の関数やコマンドが互換性のあるバージョンの CMake で実行できることを保証します。

プロジェクトを開始するにあたり、:command:`project` コマンドを使ってプロジェクトの名前を設定します。
これは全てのプロジェクトで必要なコマンドであり、:command:`cmake_minimum_required` コマンドの直後で呼び出して下さい。
のちほど説明しますが、このコマンドは他のプロジェクトのレベル情報（プログラミング言語やバージョン番号といったもの）を指定する際にも使用します。

最後のコマンドである :command:`add_executable` は、指定したソース・ファイルを使って実行形式を生成するよう CMake に指示します。

目標
----

簡単な CMake プロジェクトを作成する方法を学ぶ。

参考情報
--------

* :command:`add_executable`
* :command:`cmake_minimum_required`
* :command:`project`

編集するファイル
----------------

* ``CMakeLists.txt``

始める
------

ソース・ファイルである ``tutorial.cxx`` は ``Help/guide/tutorial/Step1`` ディレクトリにあり、平方根を計算するプログラムです。
このステップでは、このファイルを編集しません。

ソース・ファイルと同じディレクトリに ``CMakeLists.txt`` ファイルがあります。このステップでは、こちらのファイルを完成させます。
プログラム中の ``TODO 1`` から始めて ``TODO 3`` まで進めて下さい。

ビルドと実行
------------

``TODO 1`` から ``TODO 3`` まで完了したら、プロジェクトをビルドして実行する準備ができました。
まず :manual:`cmake <cmake(1)>` コマンドか、:manual:`cmake-gui <cmake-gui(1)>` を起動してプロジェクトを構成し、それから指定したビルド・ツールでプロジェクトをビルドします。

たとえば、コマンドラインから CMake のソースコード・ツリーの ``Help/guide/tutorial`` ディレクトリへ移動して、ビルドツリーに相当するディレクトリを作成します：

.. code-block:: console

  mkdir Step1_build

次に、そのディレクトリへ移動し :manual:`cmake <cmake(1)>` を起動してプロジェクトを構成し、ネィティブなビルドシステムを生成します：

.. code-block:: console

  cd Step1_build
  cmake ../Step1

そしてビルドシステムを呼び出して、実際にプロジェクトをコンパイル／リンクします：

.. code-block:: console

  cmake --build .

multi-config 対応のジェネレータ（たとえば Visual Studio）の場合は、まず次のように適切なサブディレクトリへ移動します：

.. code-block:: console

  cd Debug

最後に、ビルドした ``Tutorial`` を実行してみます：

.. code-block:: console

  Tutorial 4294967296
  Tutorial 10
  Tutorial


**注意:** お使いのシェルによっては ``Tutorial``、あるいは ``./Tutorial``、もしくは ``.\Tutorial`` が正しいコマンドラインかもしれません。
説明を簡単にするために、ここにある演習では ``Tutorial`` で統一します。

解決方法
--------

上で説明したように、ビルドして実行するために必要なことは ``CMakeLists.txt`` にある３行だけです。
最初の行は :command:`cmake_minimum_required` コマンドを使って、CMake のバージョンをセットしています:

.. raw:: html

  <details><summary>TODO 1: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 1: CMakeLists.txt
  :name: CMakeLists.txt-cmake_minimum_required
  :language: cmake
  :end-before: # set the project name and version

.. raw:: html

  </details>

CMake プロジェクトを作成するための次のステップは、:command:`project` コマンドを使ってプロジェクトの名前をセットすることです：

.. raw:: html

  <details><summary>TODO 2: （クリックして答を見る／隠す）</summary>

.. code-block:: cmake
  :caption: TODO 2: CMakeLists.txt
  :name: CMakeLists.txt-project

  project(Tutorial)

.. raw:: html

  </details>

プロジェクトを作成する最後のコマンドは :command:`add_executable` です。次のようにして呼び出します：

.. raw:: html

  <details><summary>TODO 3: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 3: CMakeLists.txt
  :name: CMakeLists.txt-add_executable
  :language: cmake
  :start-after: # add the executable
  :end-before: # TODO 3:

.. raw:: html

  </details>

演習２ - Ｃ++ 標準を指定する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CMake には、プロジェクト生成中に裏でこっそり生成される、あるいはプロジェクト・コードがセットした時になって初めて CMake で意味を持つ特殊な変数がいくつかあります。
それらの多くは、変数名が ``CMAKE_`` で始まります。
プロジェクトで変数を作成する時は、この規則を破るような変数名は避けて下さい。
このユーザがセットできる特殊な変数の二つが :variable:`CMAKE_CXX_STANDARD` と :variable:`CMAKE_CXX_STANDARD_REQUIRED` です。
これらを一緒に使用して、プロジェクトのビルドに必要なＣ++ 標準を指定できます。

目標
----

C++11 を必要とする機能をプログラムに追加する。

参考情報
--------

* :variable:`CMAKE_CXX_STANDARD`
* :variable:`CMAKE_CXX_STANDARD_REQUIRED`
* :command:`set`

編集するファイル
----------------

* ``CMakeLists.txt``
* ``tutorial.cxx``

始める
------

``Step1`` ディレクトリにあるファイルの編集の続きです。
``TODO 4`` から始めて ``TODO 6`` まで進めて下さい。

まず、``tutorial.cxx`` を編集して、C++11 を必要とする機能を追加します。
そして C++11 を要求するように ``CMakeLists.txt`` を更新して下さい。

ビルドと実行
------------

プロジェクトを再ビルドしましょう。
演習１で既にビルドツリーに相当するディレクトリを作成し CMake を起動しているので、このままビルドのステップに進んで下さい：

.. code-block:: console

  cd Step1_build
  cmake --build .

ここで、演習１と同様のコマンドで、新しくビルドした ``Tutorial`` を実行できます：

.. code-block:: console

  Tutorial 4294967296
  Tutorial 10
  Tutorial

解決方法
--------

``tutorial.cxx`` の ``atof`` を ``std::atod`` で置き換え、C++11 の機能をプロジェクトに追加するところから始まります。
これは、次のようになります：

.. raw:: html

  <details><summary>TODO 4: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/tutorial.cxx
  :caption: TODO 4: tutorial.cxx
  :name: tutorial.cxx-cxx11
  :language: c++
  :start-after: // convert input to double
  :end-before: // TODO 6:

.. raw:: html

  </details>

``TODO 5`` は単に ``#include <cstdlib>`` の文を削除するだけです。

プロジェクト・コードの中で正しいコンパイル・フラグを使うよう CMake に明示的に指示する必要があります。
特定の Ｃ++ 標準のサポートを有効にする方法の一つに :variable:`CMAKE_CXX_STANDARD` 変数を使うと云う方法があります。
このチュートリアルでは、``CMakeLists.txt`` の中で変数 :variable:`CMAKE_CXX_STANDARD` に ``11`` をセットし、変数 :variable:`CMAKE_CXX_STANDARD_REQUIRED` に ``True`` をセットしています。
必ず :command:`add_executable` コマンドの呼び出しよりも前で、変数 :variable:`CMAKE_CXX_STANDARD` をセットしているか確認して下さい。

.. raw:: html

  <details><summary>TODO 6: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 6: CMakeLists.txt
  :name: CMakeLists.txt-CXX_STANDARD
  :language: cmake
  :start-after: # specify the C++ standard
  :end-before: # configure a header file

.. raw:: html

  </details>

演習３ - バージョン番号とヘッダ・ファイルを追加する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

プロジェクトの ``CMakeLists.txt`` の中で定義されている変数を、プログラムの中で参照できると便利な場合があります。
ここではプロジェクトのバージョンをプログラムで出力させることにします。

これを実現する方法の一つにヘッダ・ファイルを利用すると云う方法があります。
まず置換を行う一個以上の変数を定義した入力用ファイルを用意します。
これらの変数は ``@VAR@`` のような特殊な構文にします。
そして :command:`configure_file` コマンドを使い、入力用のファイルを出力用ファイルにコピーして、その中にある変数を ``CMakeLists.txt`` ファイルでセットした変数 ``VAR`` の現在値で置き換えるようにします。

プログラムの中で直接バージョンを定義することは可能ですが、信頼できる単一の情報源から値を引用し、定義の重複を避けるためにも、この機能の使用を推奨します。

目標
----

プロジェクトのバージョン番号を定義して報告させる。

参考情報
--------

* :variable:`<PROJECT-NAME>_VERSION_MAJOR`
* :variable:`<PROJECT-NAME>_VERSION_MINOR`
* :command:`configure_file`
* :command:`target_include_directories`

編集するファイル
----------------

* ``CMakeLists.txt``
* ``tutorial.cxx``

始める
------

``Step1`` からファイルの編集を続けます。
``TODO 7`` から始めて ``TODO 12`` まで進めて下さい。
この演習では、``CMakeLists.txt`` の中にプロジェクトのバージョン番号を定義するところから始めます。
同じファイル内で、:command:`configure_file` コマンドを使って、指定した入力用ファイルを出力用ファイルにコピーし、入力用ファイルの中にあるいくつかの変数の値を置き換えます。

次にバージョン番号を定義し、:command:`configure_file` コマンドから渡される変数の値を受け入れる入力用ヘッダ・ファイルの ``TutorialConfig.h.in`` を作成します。

最後に ``tutorial.cxx`` を変更し、バージョン番号を出力するコードを追加して下さい。

ビルドと実行
------------

プロジェクトを再ビルドしましょう。
演習２と同様に、既にビルドツリーに相当するディレクトリを作成し CMake を起動しているので、このままビルドのステップに進んで下さい：

.. code-block:: console

  cd Step1_build
  cmake --build .

実行形式を引数無しで実行するとバージョン番号が報告されることを確認して下さい。


解決方法
--------

この演習では、バージョン番号を出力するように実行形式を改良します。
これをプログラムだけの変更で実現することは可能ですが、``CMakeLists.txt`` を使用すると、バージョン番号の定義を一意に管理できるようになります。

まず ``CMakeLists.txt`` ファイルを変更し、:command:`project` コマンドでプロジェクトの名前とバージョン番号の両方をセットします。
:command:`project` コマンドを呼び出すと、CMake は内部で ``Tutorial_VERSION_MAJOR`` と ``Tutorial_VERSION_MINOR`` の変数を定義します。

.. raw:: html

  <details><summary>TODO 7: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 7: CMakeLists.txt
  :name: CMakeLists.txt-project-VERSION
  :language: cmake
  :start-after: # set the project name and version
  :end-before: # specify the C++ standard

.. raw:: html

  </details>

そして :command:`configure_file` コマンドを使い、CMake の変数を置き換えた入力用のファイルをコピーします：

.. raw:: html

  <details><summary>TODO 8: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 8: CMakeLists.txt
  :name: CMakeLists.txt-configure_file
  :language: cmake
  :start-after: # to the source code
  :end-before: # TODO 2:

.. raw:: html

  </details>

ここでコピーした出力用ファイルは、プロジェクトのビルドツリーのディレクトリに保存されるため、そのディレクトリをインクルード・ファイルの検索パスに追加する必要があります。

**注意:** このチュートリアルでは、プロジェクトのビルド・ディレクトリとビルドツリーのディレクトリは同義として扱います。
これらは同じディレクトリであり、`bin/` ディレクトリを指しているのではありません。

ここでは :command:`target_include_directories` コマンドを使って、ビルドするターゲットがインクルード・ファイルを検索する場所を指定しています。

.. raw:: html

  <details><summary>TODO 9: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 9: CMakeLists.txt
  :name: CMakeLists.txt-target_include_directories
  :language: cmake
  :start-after: # so that we will find TutorialConfig.h

.. raw:: html

  </details>

``TutorialConfig.h.in`` ファイルはテンプレートに相当する入力用ヘッダ・ファイルです。
``CMakeLists.txt`` から :command:`configure_file` コマンドを呼び出すと、``@Tutorial_VERSION_MAJOR@`` と ``@Tutorial_VERSION_MINOR@`` が対応するプロジェクトのバージョン番号に置き換えられて、出力用ヘッダ・ファイル ``TutorialConfig.h`` に書き出されます。

.. raw:: html

  <details><summary>TODO 10: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/TutorialConfig.h.in
  :caption: TODO 10: TutorialConfig.h.in
  :name: TutorialConfig.h.in
  :language: c++

.. raw:: html

  </details>

そして ``tutorial.cxx`` を変更し、出力用ヘッダ・ファイルである ``TutorialConfig.h`` をインクルードして下さい。

.. raw:: html

  <details><summary>TODO 11: （クリックして答を見る／隠す）</summary>

.. code-block:: c++
  :caption: TODO 11: tutorial.cxx

  #include "TutorialConfig.h"

.. raw:: html

  </details>

最後に ``tutorial.cxx`` を次のように変更し、実行形式の名前とバージョン番号を出力するコードを追加して下さい：

.. raw:: html

  <details><summary>TODO 12: （クリックして答を見る／隠す）</summary>

.. literalinclude:: Step2/tutorial.cxx
  :caption: TODO 12 : tutorial.cxx
  :name: tutorial.cxx-print-version
  :language: c++
  :start-after: {
  :end-before: // convert input to double

.. raw:: html

  </details>
