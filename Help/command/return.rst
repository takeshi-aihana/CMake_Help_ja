return
------

:command:`include` したファイルやディレクトリ、または呼び出した関数から戻る。

.. code-block:: cmake

  return([PROPAGATE <var-name>...])

このコマンドが :command:`include` や :command:`find_package` コマンドで読み込んだファイルやモジュールの中で呼び出されると、現在の処理を停止して、制御を読み込み元のファイルに返します。
このコマンドが :command:`include` や :command:`find_package` コマンドで読み込でいないファイルの中で呼び出されると（たとえば ``CMakeLists.txt`` や :command:`cmake_language(DEFER)` コマンドで遅延呼び出しがスケジューリングされている場合）、親ディレクトリが存在していれば、そこに制御を返します。

関数の中で ``return()`` コマンドを呼び出すと、制御はその関数の呼び出し元に返します。
:command:`function` コマンドとは異なり、:command:`macro` コマンドの中で ``return()`` コマンドを呼び出しても正しく処理できない点に注意して下さい。

:policy:`CMP0140` というポリシーで、コマンドのオプションに関する動作を定義しています。
このポリシーに ``NEW`` を設定しない限り、``return()`` コマンドに渡された全てのオプションは無視されます。

``PROPAGATE``
  .. versionadded:: 3.25

  このオプションは、親ディレクトリや関数の呼び出し元のスコープ内で定義された ``<var-name> ...`` をセットしたり解除する。
  これは、:command:`block` コマンドを使う場合を除き、:command:`set(PARENT_SCOPE)` や :command:`unset(PARENT_SCOPE)` コマンドの呼び出しとそれぞれ同じである。

  このオプションは、:command:`block` コマンドと併用すると非常に便利である。
  ``return()`` コマンドは、``<var-name> ...`` の変数を、:command:`block` コマンドで囲んだブロック・スコープ内で伝搬する。
  :command:`function` コマンドの場合は、そのブロックに関係なく、変数はその関数の呼び出し元に確実に伝搬する。
  関数の外で ``return()`` コマンドを呼び出した場合、変数は親ファイルやディレクトリのスコープに確実に伝搬する。
  例えば：

  .. code-block:: cmake
    :caption: CMakeLists.txt

    cmake_version_required(VERSION 3.25)
    project(example)

    set(var1 "top-value")

    block(SCOPE_FOR VARIABLES)
      add_subdirectory(subDir)
      # var1 の値は "block-nested"
    endblock()

    # var1 の値は "top-value"

  .. code-block:: cmake
    :caption: subDir/CMakeLists.txt

    function(multi_scopes result_var1 result_var2)
      block(SCOPE_FOR VARIABLES)
        # 変数はブロック内でのみ伝搬し、
        # この関数の呼び出し元には伝搬しない
        #set(${result_var1} "new-value" PARENT_SCOPE)
        #unset(${result_var2} PARENT_SCOPE)

        # 変数を囲むブロックを介し
        # 変数が関数の呼び出し元に伝搬する
        set(${result_var1} "new-value")
        unset(${result_var2})
        return(PROPAGATE ${result_var1} ${result_var2})
      endblock()
    endfunction()

    set(var1 "some-value")
    set(var2 "another-value")

    multi_scopes(var1 var2)
    # ここで var1 の値は "new-value"、var2 の値はなし

    block(SCOPE_FOR VARIABLES)
      # This return() will set var1 in the directory scope that included us
      # via add_subdirectory(). The surrounding block() here does not limit
      # propagation to the current file, but the block() in the parent
      # directory scope does prevent propagation going any further.
      set(var1 "block-nested")
      return(PROPAGATE var1)
    endblock()

参考情報
^^^^^^^^

* :command:`block`
* :command:`function`
