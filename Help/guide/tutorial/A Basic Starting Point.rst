ステップ１: 基本を学ぶ出発点
============================

CMake の基本を学ぶために、どこから始めればよいでしょうか？
このステップでは、CMake の基本的な構文、CMake のコマンド、そして CMake の変数についてその一部を紹介します。
これらの概念について紹介しながら、3つの演習にとりくみ、簡単な CMake プロジェクトを生成してみます。

このステップにある演習はそれぞれ、その背景にあるもの（前提や条件）について説明するところから始めます。
そのあとで演習の目標と、参考になるいろいろなリソースについて説明します。
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
のちほど説明しますが、このコマンドは、他のプロジェクトのレベル情報（プログラミング言語やバージョン番号といったもの）を指定する際にも使用します。

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

最後はビルドした ``Tutorial`` を実行してみます：

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

Exercise 2 - Specifying the C++ Standard
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

CMake has some special variables that are either created behind the scenes or
have meaning to CMake when set by project code. Many of these variables start
with ``CMAKE_``. Avoid this naming convention when creating variables for your
projects. Two of these special user settable variables are
:variable:`CMAKE_CXX_STANDARD` and :variable:`CMAKE_CXX_STANDARD_REQUIRED`.
These may be used together to specify the C++ standard needed to build the
project.

Goal
----

Add a feature that requires C++11.

Helpful Resources
-----------------

* :variable:`CMAKE_CXX_STANDARD`
* :variable:`CMAKE_CXX_STANDARD_REQUIRED`
* :command:`set`

Files to Edit
-------------

* ``CMakeLists.txt``
* ``tutorial.cxx``

Getting Started
---------------

Continue editing files in the ``Step1`` directory. Start with ``TODO 4`` and
complete through ``TODO 6``.

First, edit ``tutorial.cxx`` by adding a feature that requires C++11. Then
update ``CMakeLists.txt`` to require C++11.

Build and Run
-------------

Let's build our project again. Since we already created a build directory and
ran CMake for Exercise 1, we can skip to the build step:

.. code-block:: console

  cd Step1_build
  cmake --build .

Now we can try to use the newly built ``Tutorial`` with same commands as
before:

.. code-block:: console

  Tutorial 4294967296
  Tutorial 10
  Tutorial

Solution
--------

We start by adding some C++11 features to our project by replacing
``atof`` with ``std::stod`` in ``tutorial.cxx``. This looks like
the following:

.. raw:: html

  <details><summary>TODO 4: Click to show/hide answer</summary>

.. literalinclude:: Step2/tutorial.cxx
  :caption: TODO 4: tutorial.cxx
  :name: tutorial.cxx-cxx11
  :language: c++
  :start-after: // convert input to double
  :end-before: // TODO 6:

.. raw:: html

  </details>

To complete ``TODO 5``, simply remove ``#include <cstdlib>``.

We will need to explicitly state in the CMake code that it should use the
correct flags. One way to enable support for a specific C++ standard in CMake
is by using the :variable:`CMAKE_CXX_STANDARD` variable. For this tutorial, set
the :variable:`CMAKE_CXX_STANDARD` variable in the ``CMakeLists.txt`` file to
``11`` and :variable:`CMAKE_CXX_STANDARD_REQUIRED` to ``True``. Make sure to
add the :variable:`CMAKE_CXX_STANDARD` declarations above the call to
:command:`add_executable`.

.. raw:: html

  <details><summary>TODO 6: Click to show/hide answer</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 6: CMakeLists.txt
  :name: CMakeLists.txt-CXX_STANDARD
  :language: cmake
  :start-after: # specify the C++ standard
  :end-before: # configure a header file

.. raw:: html

  </details>

Exercise 3 - Adding a Version Number and Configured Header File
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sometimes it may be useful to have a variable that is defined in your
``CMakelists.txt`` file also be available in your source code. In this case, we
would like to print the project version.

One way to accomplish this is by using a configured header file. We create an
input file with one or more variables to replace. These variables have special
syntax which looks like ``@VAR@``.
Then, we use the :command:`configure_file` command to copy the input file to a
given output file and replace these variables with the current value of ``VAR``
in the ``CMakelists.txt`` file.

While we could edit the version directly in the source code, using this
feature is preferred since it creates a single source of truth and avoids
duplication.

Goal
----

Define and report the project's version number.

Helpful Resources
-----------------

* :variable:`<PROJECT-NAME>_VERSION_MAJOR`
* :variable:`<PROJECT-NAME>_VERSION_MINOR`
* :command:`configure_file`
* :command:`target_include_directories`

Files to Edit
-------------

* ``CMakeLists.txt``
* ``tutorial.cxx``

Getting Started
---------------

Continue to edit files from ``Step1``. Start on ``TODO 7`` and complete through
``TODO 12``. In this exercise, we start by adding a project version number in
``CMakeLists.txt``. In that same file, use :command:`configure_file` to copy a
given input file to an output file and substitute some variable values in the
input file content.

Next, create an input header file ``TutorialConfig.h.in`` defining version
numbers which will accept variables passed from :command:`configure_file`.

Finally, update ``tutorial.cxx`` to print out its version number.

Build and Run
-------------

Let's build our project again. As before, we already created a build directory
and ran CMake so we can skip to the build step:

.. code-block:: console

  cd Step1_build
  cmake --build .

Verify that the version number is now reported when running the executable
without any arguments.

Solution
--------

In this exercise, we improve our executable by printing a version number.
While we could do this exclusively in the source code, using ``CMakeLists.txt``
lets us maintain a single source of data for the version number.

First, we modify the ``CMakeLists.txt`` file to use the
:command:`project` command to set both the project name and version number.
When the :command:`project` command is called, CMake defines
``Tutorial_VERSION_MAJOR`` and ``Tutorial_VERSION_MINOR`` behind the scenes.

.. raw:: html

  <details><summary>TODO 7: Click to show/hide answer</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 7: CMakeLists.txt
  :name: CMakeLists.txt-project-VERSION
  :language: cmake
  :start-after: # set the project name and version
  :end-before: # specify the C++ standard

.. raw:: html

  </details>

Then we used :command:`configure_file` to copy the input file with the
specified CMake variables replaced:

.. raw:: html

  <details><summary>TODO 8: Click to show/hide answer</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 8: CMakeLists.txt
  :name: CMakeLists.txt-configure_file
  :language: cmake
  :start-after: # to the source code
  :end-before: # TODO 2:

.. raw:: html

  </details>

Since the configured file will be written into the project binary
directory, we must add that directory to the list of paths to search for
include files.

**Note:** Throughout this tutorial, we will refer to the project build and
the project binary directory interchangeably. These are the same and are not
meant to refer to a `bin/` directory.

We used :command:`target_include_directories` to specify
where the executable target should look for include files.

.. raw:: html

  <details><summary>TODO 9: Click to show/hide answer</summary>

.. literalinclude:: Step2/CMakeLists.txt
  :caption: TODO 9: CMakeLists.txt
  :name: CMakeLists.txt-target_include_directories
  :language: cmake
  :start-after: # so that we will find TutorialConfig.h

.. raw:: html

  </details>

``TutorialConfig.h.in`` is the input header file to be configured.
When :command:`configure_file` is called from our ``CMakeLists.txt``, the
values for ``@Tutorial_VERSION_MAJOR@`` and ``@Tutorial_VERSION_MINOR@`` will
be replaced with the corresponding version numbers from the project in
``TutorialConfig.h``.

.. raw:: html

  <details><summary>TODO 10: Click to show/hide answer</summary>

.. literalinclude:: Step2/TutorialConfig.h.in
  :caption: TODO 10: TutorialConfig.h.in
  :name: TutorialConfig.h.in
  :language: c++

.. raw:: html

  </details>

Next, we need to modify ``tutorial.cxx`` to include the configured header file,
``TutorialConfig.h``.

.. raw:: html

  <details><summary>TODO 11: Click to show/hide answer</summary>

.. code-block:: c++
  :caption: TODO 11: tutorial.cxx

  #include "TutorialConfig.h"

.. raw:: html

  </details>

Finally, we print out the executable name and version number by updating
``tutorial.cxx`` as follows:

.. raw:: html

  <details><summary>TODO 12: Click to show/hide answer</summary>

.. literalinclude:: Step2/tutorial.cxx
  :caption: TODO 12 : tutorial.cxx
  :name: tutorial.cxx-print-version
  :language: c++
  :start-after: {
  :end-before: // convert input to double

.. raw:: html

  </details>
