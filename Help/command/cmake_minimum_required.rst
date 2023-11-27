cmake_minimum_required
----------------------

CMake の最低バージョンを要求する。

.. code-block:: cmake

  cmake_minimum_required(VERSION <min>[...<policy_max>] [FATAL_ERROR])

.. versionadded:: 3.12
  ``<policy_max>`` オプションの追加。

プロジェクトに最低限必要な CMake のバージョンを設定します。
加えて、以下に説明するプロジェクトのポリシーの設定も更新します。

``<min>`` とオプションの ``<policy_max>`` には、それぞれ CMake のバージョンを ``major.minor[.patch[.tweak]]`` 形式で指定します。``...`` はそのままの文字列（リテラル）を表します。

実行中の CMake のバージョンが、指定した ``<min>`` よりも小さい場合はプロジェクトの生成を停止してエラーを報告します。
オプションの ``<policy_max>`` を指定した場合は、それが `ポリシーの設定`_ で説明する「**ポリシーの設定**」に影響するため、少なくともバージョンが ``<min>`` である :manual:`cmake(1)` コマンドを実行する必要があります。
もし実行中の CMake のバージョンが 3.12 より古い場合は、``...`` がバージョン・コンポーネントの区切り文字として認識され、その結果 ``...<max>`` の部分が無視されて、``<min>`` バージョンに基づくポリシーのうち、バージョン 3.12 より前の動作（機能）が保証されます。

このコマンドは、CMake 変数の :variable:`CMAKE_MINIMUM_REQUIRED_VERSION` に ``<min>`` をセットします。

オプションの ``FATAL_ERROR`` も受け取りますが、CMake バージョン 2.6 以上では無視されます。
CMake のバージョン 2.4 以下の場合に警告ではなく、エラーとして失敗するように指定して下さい。

.. note::
  たとえ :command:`project` コマンドを呼び出す前であっても、プロジェクト最上位にある ``CMakeLists.txt`` ファイルの先頭で、この ``cmake_minimum_required()`` コマンドを呼び出して下さい。
  動作に影響を与える可能性があるその他のコマンドを呼び出す前に、CMake のバージョンとポリシーの設定を確立しておくことが重要です。
  ポリシーの :policy:`CMP0000` も参照して下さい。

  :command:`function` の中で ``cmake_minimum_required()`` コマンドを呼び出すと、それを呼び出した時に、関数スコープに対する一部の効果が制限されます。
  たとえば、CMake 変数の :variable:`CMAKE_MINIMUM_REQUIRED_VERSION` には何もセットされません。
  ただし、関数には独自のポリシー・スコープが適用されないので、結果的に関数の呼び出し元のポリシーにも影響を与えることになります（以下、参照）。
  すなわち、呼び出し元のスコープに影響を与えるものと与えないものが混在することになるので、一般的に任意の :command:`function` の中で ``cmake_minimum_required()`` コマンドを呼び出すことは推奨されません。

.. _`Policy Settings`:

ポリシーの設定
^^^^^^^^^^^^^^

``cmake_minimum_required(VERSION)`` コマンドは暗黙的に :command:`cmake_policy(VERSION)` コマンドを呼び出して、指定した範囲の CMake バージョンに対して、プロジェクトの構成を生成します。
実行中の CMake のバージョンに準じ、``<min>`` （または指定した場合は ``<max>``）以前のバージョンで導入されたポリシーは、すべて ``NEW`` の機能を使うようにセットされます。
そのあとのバージョンで導入されたポリシーはすべて解除されます。
この仕組みは、特定のバージョンの CMake が推奨する動作（機能）を効果的に要求し、新しいバージョンの CMake が提供する新しいポリシーについて警告するように指示します。

``<min>`` にバージョン 2.4 よりも大きいバージョンを指定すると、暗黙的に次のコマンドを呼び出します：

.. code-block:: cmake

  cmake_policy(VERSION <min>[...<max>])

この呼び出しにより、指定したバージョンの範囲に基づく CMake のポリシーをセットします。
``<min>`` がバージョン 2.4 以下の場合は、暗黙的に次のコマンドを呼び出します：

.. code-block:: cmake

  cmake_policy(VERSION 2.4[...<max>])

この呼び出しは CMake のバージョン 2.4 以下と互換性のある動作（機能）を有効にします。

.. include:: DEPRECATED_POLICY_VERSIONS.txt

参考情報
^^^^^^^^

* :command:`cmake_policy`
