block
-----

.. versionadded:: 3.25

専用の変数 および／または ポリシーのスコープを使ってコマンドのグループを評価する。

.. code-block:: cmake

  block([SCOPE_FOR [POLICIES] [VARIABLES] ] [PROPAGATE <var-name>...])
    <commands>
  endblock()

この ``block()`` コマンドから :command:`endblock` の間にある全てのコマンドは呼び出されることなく記録されます。
:command:`endblock` が評価されると、記録したコマンドのリストが要求されたスコープの中で呼び出され、この ``block()`` コマンドが生成した全てのスコープが削除されます。

``SCOPE_FOR``
  作成するスコープを指定する。

  ``POLICIES``
    新しいスコープを一つ作成する。
    これは、ブロック・スコープから外れる時に自動の :command:`cmake_policy(POP)` を使う  :command:`cmake_policy(PUSH)` と等価である。

  ``VARIABLES``
    新しい変数のスコープを作成する。

  もし ``SCOPE_FOR`` が指定されていなかったら、これは以下と等価である：

  .. code-block:: cmake

    block(SCOPE_FOR VARIABLES POLICIES)

``PROPAGATE``
  :command:`block` コマンドで変数のスコープを作成したら、このオプションは親スコープ内で指定した変数をセットしたり解除する。
  これは :command:`set(PARENT_SCOPE)` や :command:`unset(PARENT_SCOPE)` のコマンドと等価である。

  .. code-block:: cmake

    set(var1 "INIT1")
    set(var2 "INIT2")

    block(PROPAGATE var1 var2)
      set(var1 "VALUE1")
      unset(var2)
    endblock()

    # ここでは var1 = VALUE1、var2 = ''
 
  このオプションは、変数のスコープを作成する時にだけ指定できる。それ以外はエラーとなる。

この ``block()`` コマンドが :command:`foreach` や :command:`while` コマンドが生成したループ・ブロックの中にある時は、:command:`break` と :command:`continue` コマンドをそのブロックの中でそれぞれ呼び出すことは可能です。

.. code-block:: cmake

  while(TRUE)
    block()
       ...
       # break() コマンドで while() コマンドのループ処理を終了する
       break()
    endblock()
  endwhile()


参考情報
^^^^^^^^

* :command:`endblock`
* :command:`return`
* :command:`cmake_policy`
