message
-------

メッセージをログする。

概要
^^^^

.. parsed-literal::

  `一般的なメッセージ`_
    message([<mode>] "message text" ...)

  `テストした結果を報告するメッセージ`_
    message(<checkState> "message text" ...)

  `Configure ログのメッセージ`_
    message(CONFIGURE_LOG <text>...)

一般的なメッセージ
^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  message([<mode>] "message text" ...)

ログに ``"message text"`` を記録します。
複数の ``"message text" ...`` を渡すと、区切り文字を挿入せずに一個のメッセージに連結して記録します。

``<mode>`` オプションでログするメッセージのレベル（種類）を指定します。このオプションはメッセージの処理方法に影響します：

``FATAL_ERROR``
  CMake のエラー。
  ビルド処理やファイルの生成を停止する。

  :manual:`cmake(1)` コマンドは 0 以外の「:ref:`終了コード <CMake Exit Code>`」を返す。

``SEND_ERROR``
  CMake のエラー。
  ビルド処理は続行するが、ファイルの生成はスキップする。

``WARNING``
  CMake のワーニング。
  ビルド処理は続行する。

``AUTHOR_WARNING``
  CMake のワーニング（dev）。
  ビルド処理は続行する。

``DEPRECATION``
  CMake 変数の :variable:`CMAKE_ERROR_DEPRECATED` または :variable:`CMAKE_WARN_DEPRECATED` が有効な場合は、CMake の非推奨エラーまたはワーニングをログし、それ以外のメッセージはログしない。

(none) または ``NOTICE``
  ユーザの注意を引くために、重要なメッセージを標準エラー出力にログする。

``STATUS``
  ユーザが関心を持つであろう情報をログする。
  理想的には、1行分の簡潔なメッセージにすること。

``VERBOSE``
  ユーザを対象とした詳細な情報をログする。
  これらのメッセージは、ほとんどの場合は直接ビルドとは関係のない追加の情報を提供するためのもので、開発者にとっては何が起こっているのか深い洞察が必要なケースに役立つ詳細情報になる場合がある。

``DEBUG``
  プロジェクトをビルドするだけのユーザではなく、プロジェクトに取り組んでいる開発者を対象とした詳細な情報をログする。
  これらのメッセージは、プロジェクトをビルドしているユーザには興味あるものではなく、多くの場合は内部にある実装の詳細に密接に関係する情報である。

``TRACE``
  非常に低レベルの実装詳細を含んだきめの細かい情報をログする。
  このレベルのメッセージは、通常は一時的な情報であり、プロジェクトのリリースやパッケージ化などを行う前に削除されることが想定されている。

.. versionadded:: 3.15
  ``NOTICE`` と ``VERBOSE`` と ``DEBUG`` と ``TRACE`` レベルが追加された。

:manual:`CMake コマンドライン <cmake(1)>` は ``STATUS`` から ``TRACE`` レベルのメッセージを、``"message text"`` の前に2個のハイフンとスペース（"``--`` "）を付けて、標準出力にログします。
それ以外のレベルのメッセージは全て標準エラー出力にログされ、先頭にハイフン（"``--``"）は付与しません。
:manual:`CMake GUI インタフェース <cmake-gui(1)>` はログ表示エリアに全てのメッセージを表示します。
:manual:`CMake Curses インタフェース <ccmake(1)>` は、ステータス行に ``STATUS`` から ``TRACE`` レベルのメッセージを1個ずつ表示し、その他のメッセージはポップアップ・ダイアログに表示します。
これらの CMake ツールで :option:`--log-level <cmake --log-level>` オプションを使うと、どのメッセージをログするかフィルタリングできます。

.. versionadded:: 3.17
  CMake の実行時にログのレベルを保持するための CMake 変数 :variable:`CMAKE_MESSAGE_LOG_LEVEL` が導入された。
  コマンドライン・オプションはキャッシュ変数よりも優先されることに注意すること。

.. versionadded:: 3.16
  ``NOTICE`` 以下のレベルのメッセージの先頭に CMake 変数 :variable:`CMAKE_MESSAGE_INDENT` （:ref:`リスト <CMake Language Lists>` 型）の内容をログするようになった（リストの要素を連結することで単一の文字列にする）。
  ``STATUS`` から ``TRACE`` レベルのメッセージの場合、この内容はハイフン（"``--``"）の後ろに挿入される。

.. versionadded:: 3.17
  ``NOTICE`` 以下のレベルのメッセージの先頭に ``[some.context.example]`` 形式のコンテキストを挿入できるようになった。
  "``[``" と "``]``" の中にある文字列は、CMake 変数 :variable:`CMAKE_MESSAGE_CONTEXT` （:ref:`リスト <CMake Language Lists>` 型）の要素をドット文字（"``.``"）で連結したもの。
  このコンテキストは先頭のハイフン（"``--``"）とインデントされたメッセージとの間に挿入される。
  デフォルトではコンテキストを挿入しない。
  そのため、オプション付きで :option:`cmake --log-context` コマンドラインを実行するか、CMake 変数の :variable:`CMAKE_MESSAGE_CONTEXT_SHOW` を ``TRUE`` にすることでコンテキストの挿入を明示的にすること。
  CMake 変数 :variable:`CMAKE_MESSAGE_CONTEXT` に記載した使用例を参照のこと。

CMake のワーニングやエラーのメッセージは単純なマークアップ言語を使ってログしています。
インデントがないメッセージは、改行文字で区切られ、改行した段落の中で整形されています。
インデントがあるメッセージは事前に整形されたものとみなされます。


テストした結果を報告するメッセージ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.17

CMake がログするメッセージの典型的なパタンは、何か変数やファイルの状態をテストする旨のメッセージのあとに、その結果を報告する別のメッセージが続くというものです。
例えば：

.. code-block:: cmake

  message(STATUS "Looking for someheader.h") # テストを開始するメッセージ
  #... someheader.h があるかテストし、その結果を checkSuccess に格納する
  if(checkSuccess)
    message(STATUS "Looking for someheader.h - found")   # テストした結果のメッセージ
  else()
    message(STATUS "Looking for someheader.h - not found") # テストした結果のメッセージ
  endif()

このようなログは ``message()`` コマンドの ``CHECK_...`` キーワードを使用すると、より安全で便利になります：

.. code-block:: cmake

  message(<checkState> "message" ...)

``<checkState>``  には、以下のいずれかを指定します： 

  ``CHECK_START``
    これから実施するテストの簡単なメッセージをログする。

  ``CHECK_PASS``
    テストが成功したときの結果をログする。

  ``CHECK_FAIL``
    テストが失敗したときの結果をログする。

これらのキーワードを使ってテストした結果を記録する時は、テストを開始した直後から、いくつかの区切り文字と ``CHECK_PASS`` または ``CHECK_FAIL`` のキーワード、そしてメッセージの本体からなるログを繰り返します。
なお、これらのメッセージは常に ``STATUS`` レベルです。

テストはネストすることができ、その場合は ``CHECK_START`` 毎に対応した ``CHECK_PASS`` または ``CHECK_FAIL`` の結果が一個だけログされるようにします。
CMake 変数の :variable:`CMAKE_MESSAGE_INDENT` もネストしたテストのログで利用できます。
例えば：

.. code-block:: cmake

  message(CHECK_START "Finding my things") # 親テストの開始を伝えるメッセージ
  list(APPEND CMAKE_MESSAGE_INDENT "  ")   # インデントを追加する
  unset(missingComponents)

  message(CHECK_START "Finding partA")     # 子テストAの開始を伝えるメッセージ
  # ... テストする
  message(CHECK_PASS "found")              # 子テストAの結果を伝えるメッセージ

  message(CHECK_START "Finding partB")     # 子テストBの開始を伝えるメッセージ
  # ... テストする
  list(APPEND missingComponents B)
  message(CHECK_FAIL "not found")          # 子テストBの結果を伝えるメッセージ

  list(POP_BACK CMAKE_MESSAGE_INDENT)      # インデントを削除する
  if(missingComponents)
    message(CHECK_FAIL "missing components: ${missingComponents}") # 親テストの結果を伝えるメッセージ
  else()
    message(CHECK_PASS "all components found")                     # 親テストの結果を伝えるメッセージ
  endif()

この ``message()`` コマンドのログは次のようになります::

  -- Finding my things
  --   Finding partA
  --   Finding partA - found
  --   Finding partB
  --   Finding partB - not found
  -- Finding my things - missing components: B

configure ログのメッセージ
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.26

.. code-block:: cmake

  message(CONFIGURE_LOG <text>...)

``<text>`` を使って「:ref:`configure スクリプトのログ・イベント <message configure-log event>` 」をログします。
慣例により、``<text>`` に複数行のメッセージを含める場合は、先頭行のメッセージはイベントのサマリにして下さい。

このモードの目的は ``configure`` スクリプトによるシステムの調査やキャッシュ変数を併用して一回しか実行しない操作をログすることですが、自動的にログしてくれる :command:`try_compile` や :command:`try_run` コマンドでは実行されないので注意が必要です。
また、プロジェクトで CMake を実行するたびに、このモードを呼び出すようなことは避けるようにして下さい。
例えば：

.. code-block:: cmake

  if (NOT DEFINED MY_CHECK_RESULT)
    # configure の出力で、テストのサマリをログする
    message(CHECK_START "My Check")

    # ... システムの調査を実施する（例えば、テスト用のプログラムを使うなどして ...）

    # 調査結果をキャッシュしておくので、再びシステムの調査を実行することはない
    set(MY_CHECK_RESULT "${MY_CHECK_RESULT}" CACHE INTERNAL "My Check")

    # CONFIGURE_LOG モードでキャシュした結果をログする
    message(CONFIGURE_LOG
      "My Check Result: ${MY_CHECK_RESULT}\n"
      "${details}"
    )

    # configure の出力の中に調査結果をログする
    if(MY_CHECK_RESULT)
      message(CHECK_PASS "passed")
    else()
      message(CHECK_FAIL "failed")
    endif()
  endif()

未だプロジェクトで ``configure`` が実行されていない場合は、このコマンドは何も行いません（たとえば :ref:`cmake -P <Script Processing Mode>` を実行した時など）。

参考情報
^^^^^^^^

* :command:`cmake_language(GET_MESSAGE_LOG_LEVEL)`
