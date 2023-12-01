cmake_policy
------------

CMake プロジェクトのポリシーを管理する。
これまで定義されたポリシーとその設定については :manual:`cmake-policies(7)` のマニュアルを参照のこと。

CMake が進化するにつれて、バグを修正したり、既存の機能を改善するための変更が必要になることがある。
「CMake のポリシー・メカニズム」は、新しいバージョンの CMake で動作が変更になった時に、古いバージョンで生成したプロジェクトのビルドを継続できるよう設計されている。
動作の変更、すなわち「**ポリシー**」が新しくなると ``CMP<NNNN>`` という識別子が付与される（ ``<NNNN>`` は整数のインデックス）。
それぞれのポリシーに関連付けられたドキュメントには、その動作（機能）が ``OLD`` なのか ``NEW`` なのかの分類と、ポリシーが導入された理由が説明されている。
CMake のプロジェクトは希望する機能を選択するためにポリシーを設定できる。
CMake は、プロジェクトでどの機能を使用するのかを知る必要がある時、そのポリシーの設定を確認する。
ポリシーが設定されていない場合は ``OLD`` のカテゴリに属す動作が期待されていると想定し、ポリシーを設定するように警告が発せられる。

CMake バージョンごとのポリシー
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

この ``cmake_policy`` コマンドを使用して、そのポリシーが ``OLD`` の機能なのか、または ``NEW`` の機能なのかをセットします。
ポリシーを一つ一つセットすることも可能ですが、CMake のバージョンに基づいて、いくつかのポリシーをセットすることをおすすめします：

.. signature:: cmake_policy(VERSION <min>[...<max>])
  :target: VERSION

.. versionadded:: 3.12
  オプション ``<max>`` （最大バージョンの指定）が追加されました。

``<min>`` とオプションの ``<max>`` はそれぞれ ``major.minor[.patch[.tweak]]`` の形式の CMake のバージョンを表し、``...`` はそのままの文字列（リテラル）を表します。
``<min>`` のバージョンには、最小値として ``2.4``、最大値として実行する CMake のバージョンを指定して下さい。
``<max>`` のバージョンを指定する場合は、最小値は ``<min>`` のバージョンですが、実行する CMake よりも大きいバージョンも指定できます。
もし実行する CMake のバージョンが 3.12 より古い場合は、``...`` がバージョン・コンポーネントの区切り文字として認識され、その結果 ``...<max>`` の部分が無視されて、``<min>`` バージョンに基づくポリシーのうち、バージョン 3.12 より前の機能（動作）が保証されます。

これは、このポリシーで生成した CMake のビルド構成が、指定された範囲の CMake バージョンに対応したコードで記述されていることを意味します。
この時、``<min>`` （または、指定していれば ``<max>``）以前のバージョンで導入されたすべてのポリシーは ``NEW`` の機能として分類されます。
そして ``<min>`` 以降のバージョンで導入されたすべてのポリシーが解除されます（ただし、CMake 変数の :variable:`CMAKE_POLICY_DEFAULT_CMP<NNNN>` でデフォルトのポリシーがセットされている場合は除く）。
これは、特定の CMake バージョンで推奨される機能を効果的に要求し、新しいバージョンの CMake に対して新しいポリシーに対する警告を出すよう指示します。

なお :command:`cmake_minimum_required(VERSION)` コマンドは暗黙的に、この ``cmake_policy(VERSION)`` コマンドを呼び出していることに注意して下さい。

.. include:: DEPRECATED_POLICY_VERSIONS.txt

ポリシーを明示的にセットする
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. signature:: cmake_policy(SET CMP<NNNN> NEW|OLD)
  :target: SET

これは、指定したポリシーに対して ``OLD`` または ``NEW`` の機能を使用するよう CMake に指示します。
この時、指定したポリシーで古い機能に依存しているプロジェクトは、そのポリシーの状態を ``OLD`` にセットすると、ポリシーに対する警告が表示されなくなります。
あるいは、そのプロジェクトを新しい機能で動作するように修正してから、そのポリシーの状態を ``NEW`` にセットすると、同様に警告が表示されなくなります。

.. include:: ../policy/DEPRECATED.txt

ポリシーを確認する
^^^^^^^^^^^^^^^^^^

.. signature:: cmake_policy(GET CMP<NNNN> <variable>)
  :target: GET

これは、指定したポリシーが ``OLD`` の機能にセットされているのか、``NEW`` の機能にセットされいるのかをチェックします。
ポリシーがセットされている場合は、``<variable>`` に ``OLD`` または ``NEW`` がセットされ、それ以外は空です。

CMake のポリシー・スタック
^^^^^^^^^^^^^^^^^^^^^^^^^^

CMake は任意のスタック上にポリシーの設定を保存しているので、``cmake_policy`` コマンドによる変更はスタックにある最上位にのみ影響します。
ポリシー・スタックの新しいエントリは、サブディレクトリ単位で自動的に管理されます。
また CMake は、:command:`include` と :command:`find_package`  コマンドでロードしたスクリプトの新しいエントリも管理します（ただし、``NO_POLICY_SCOPE`` オプションを追加して呼び出した場合は除く）。
:policy:`CMP0011` のポリシーも参照して下さい。
``cmake_policy`` コマンドは、ポリシー・スタック上で独自のエントリを管理するためのインタフェースも提供しています：

.. signature:: cmake_policy(PUSH)
  :target: PUSH

  ポリシー・スタック上に新しいエントリを一つ生成する。

.. signature:: cmake_policy(POP)
  :target: POP

  ``cmake_policy(PUSH)`` コマンドで生成したポリシー・スタック上の最後のエントリを一つ削除する。

エントリを削除するには ``PUSH`` 操作に対応する ``POP`` 操作が必要です。
このインタフェースは、ポリシーを「一時的に」変更する際に便利です。
:command:`cmake_minimum_required(VERSION)` や :command:`cmake_policy(VERSION)`、あるいは :command:`cmake_policy(SET)` といったコマンドは、ポリシー・スタックの現在のトップにのみ影響します。

.. versionadded:: 3.25
  :command:`block(SCOPE_FOR POLICIES)` コマンドが、ポリシー・スタックを管理するもっとも柔軟で安全な方法を提供するようになりました。
  POP 操作はブロックのスコープから出ると自動的に実施されるので、:command:`return` コマンドの前に :command:`cmake_policy(POP)` コマンドを呼び出す必要はなくなりました。

  .. code-block:: cmake

    # cmake_policy() コマンドを使ってポリシー・スタックを管理する
    function(my_func)
      cmake_policy(PUSH)
      cmake_policy(SET ...)
      if (<cond1>)
        ...
        cmake_policy(POP)
        return()
      elseif(<cond2>)
        ...
        cmake_policy(POP)
        return()
      endif()
      ...
      cmake_policy(POP)
    endfunction()

    # block()/endblock() を使ってポリシー・スタックを管理する
    function(my_func)
      block(SCOPE_FOR POLICIES)
        cmake_policy(SET ...)
        if (<cond1>)
          ...
          return()
        elseif(<cond2>)
          ...
          return()
        endif()
        ...
      endblock()
    endfunction()

:command:`function` や :command:`macro` といったコマンドで新しいコマンドを作成した時にポリシーの設定を記録しておき、実際にそのコマンドを呼び出した時に、記録しておいたポリシーを使います。
``function()`` や ``macro()`` の中でポリシーの設定を変更した場合、その変更は直近のスタック・エントリに到達するまで、呼び出し元を介して自動的に伝搬されます。

参考情報
^^^^^^^^

* :command:`cmake_minimum_required`
