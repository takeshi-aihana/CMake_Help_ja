function
--------

あとでコマンドとして呼び出す「関数」の定義の記録を開始する。

.. code-block:: cmake

  function(<name> [<arg1> ...])
    <commands>
  endfunction()

``<arg1>``, ... などの引数を受け取る ``<name>`` という名前の「関数」の定義を開始します。
この定義の中にある ``<commands>`` が記録されます。
これらのコマンドは、関数を呼び出すまで実行されません。

従来どおり :command:`endfunction` コマンドでは ``<name>`` を参照できます。
その場合、``function`` コマンドの引数をそのまま繰り返す必要があります。

定義した関数は新しいスコープを作成します： 詳細は  :command:`set(var PARENT_SCOPE)` を参照して下さい。

関数の内部のポリシーについては :command:`cmake_policy()` コマンドを参照して下さい。

CMake が提供する関数とマクロの違いについては :command:`macro()` コマンドを参照して下さい。

関数の呼び出し
^^^^^^^^^^^^^^

関数の呼び出しは大文字と小文字を区別しません。
例えば：

.. code-block:: cmake

  function(foo)
    <commands>
  endfunction()

この関数は次のいずれかの方法で呼び出すことができます：

.. code-block:: cmake

  foo()
  Foo()
  FOO()
  cmake_language(CALL foo)

ただし大小文字にかかわらず、関数の定義にある ``<name>`` をそのまま使うことを強く推奨します。
また ``<name>`` は、全て小文字にするのが通例になっています。

.. versionadded:: 3.18
  :command:`cmake_language(CALL ...)` コマンドで関数を呼び出すことができるようになった。

関数の引数
^^^^^^^^^^

関数を呼び出すと、まず ``<command>`` に含まれている仮引数（``${arg1}``, ``${arg2}``, ``${arg3}``, ...）を、実際に渡された実引数で置き換えてから、通常のコマンドとして呼び出します。

さらに、引数の個数が格納される ``ARGC`` 変数や、実際に引数に渡された値がそれぞれ格納された ``ARGV0``、``ARGV1``、``ARGV2``、... などの変数も参照できます。
これにより、オプションの引数を利用した関数を簡単に作成できます。

また ``ARGV`` 変数は関数に渡された全ての引数を要素とする :ref:`リスト <CMake Language Lists>` を保持し、 ``ARGN`` 変数は関数に渡された引数のうち「必須ではない」全ての引数を要素とする :ref:`リスト <CMake Language Lists>` を保持しています。
``ARGC`` （引数の総数）を超えた ``ARGV#`` を参照すると、未定義の結果になります。
``ARGC`` が ``#`` よりも大きいかどうかを確認することは、``ARGV#`` が関数に渡された追加の引数であることを確認する唯一の方法です。

参考情報
^^^^^^^^

* :command:`cmake_parse_arguments`
* :command:`endfunction`
* :command:`return`
