macro
-----

あとでコマンドとして呼び出す「マクロ」定義の記録を開始する。

.. code-block:: cmake

  macro(<name> [<arg1> ...])
    <commands>
  endmacro()

``<arg1>``, ... などの引数を受け取る ``<name>`` という名前の「マクロ」の定義を開始します。
:command:`endmacro()` まで記録したコマンド列は、このマクロを呼び出すまで実行されません。

従来どおり :command:`endmacro` コマンドでは ``<name>`` を参照できます。
その場合、``macro`` コマンドの引数をそのまま繰り返す必要があります。

マクロ内の動作ポリシーについては :command:`cmake_policy()` コマンドを参照して下さい。

CMake で「関数」と「マクロ」の違いについては「:ref:`Macro vs Function`」のセクションを参照して下さい。

マクロの呼び出し
^^^^^^^^^^^^^^^^

マクロの呼び出しは大文字と小文字を区別しません。
例えば：

.. code-block:: cmake

  macro(foo)
    <commands>
  endmacro()

このマクロは次のいずれかの方法で呼び出すことができます：

.. code-block:: cmake

  foo()
  Foo()
  FOO()
  cmake_language(CALL foo)

ただし大小文字にかかわらず、マクロの定義にある ``<name>`` をそのまま使うことを強く推奨します。
また ``<name>`` は、全て小文字にするのが通例になっています。

.. versionadded:: 3.18
  :command:`cmake_language(CALL ...)` コマンドでマクロを呼び出すことができるようになった。

マクロの引数
^^^^^^^^^^^^

任意のマクロを呼び出すと、まず ``<commands>`` に含まれている仮引数（``${arg1}``, ``${arg2}``, ``${arg3}``, ...）を、実際に渡された実引数で置き換えてから、通常のコマンドとして呼び出します。

さらに、引数の個数が格納される ``${ARGC}`` 変数や、実際に引数に渡された値がそれぞれ格納された ``${ARGV0}``、``${ARGV1}``、``${ARGV2}``、... などの変数も参照できます。
これにより、オプションの引数を利用したマクロを簡単に作成できます。

また ``${ARGV}`` 変数はマクロに渡された全ての引数を要素とする :ref:`リスト <CMake Language Lists>` を保持し、 ``${ARGN}`` 変数はマクロに渡された引数のうち「必須ではない」全ての引数を要素とする :ref:`リスト <CMake Language Lists>` を保持しています。
``${ARGC}`` （引数の総数）を超えた ``${ARGV#}`` を参照すると、未定義の結果になります。
``${ARGC}`` が ``#`` よりも大きいかどうかを確認することは、``${ARGV#}`` がマクロに渡された追加の引数であることを確認する唯一の方法です。

.. _`Macro vs Function`:

マクロ vs 関数
^^^^^^^^^^^^^^

``macro`` コマンドは :command:`function` コマンドによく似ています。
それにもかかわらず、いくつか重要な違いがあります。

まず、関数では ``ARGN``、``ARGC``、``ARGV``、そして ``ARGV0`` や ``ARGV1``、などは CMake 変数として扱われます。
これに対してマクロでは、Ｃ言語のプリプロセッサがマクロを扱う場合とよく似た「文字列の置換」になります。
この違いは「:ref:`Argument Caveats`」のセクションで説明するように、いろいろな場面で影響があります。

マクロと関数のもう一つの違いは制御フロー（*Control Flow*）にあります。
関数は、呼び出し命令から関数の本体に制御が移されて実行されます。
一方、マクロは呼び出し命令がマクロの本体に置き換えられて実行されます。
そのため、もしマクロの本体に :command:`return()` コマンドがあると、マクロの実行が完了するだけにとどまりません。
つまり、制御がマクロを呼び出したスコープから出てしまうということです。
この違いによる混乱を避けるため、マクロの中で :command:`return()` コマンドを呼び出さないことを強く推奨します。

また関数とは異なり、マクロの本体で :variable:`CMAKE_CURRENT_FUNCTION`、:variable:`CMAKE_CURRENT_FUNCTION_LIST_DIR`、 :variable:`CMAKE_CURRENT_FUNCTION_LIST_FILE`、 :variable:`CMAKE_CURRENT_FUNCTION_LIST_LINE` といった CMake 変数は利用できません。

.. _`Argument Caveats`:

引数で注意すること
^^^^^^^^^^^^^^^^^^

Since ``ARGN``, ``ARGC``, ``ARGV``, ``ARGV0`` etc. are not variables, you will NOT be able to use commands like

.. code-block:: cmake

 if(ARGV1) # ARGV1 is not a variable 
 if(DEFINED ARGV2) # ARGV2 is not a variable
 if(ARGC GREATER 2) # ARGC is not a variable
 foreach(loop_var IN LISTS ARGN) # ARGN is not a variable

In the first case, you can use ``if(${ARGV1})``.
In the second and third case, the proper way to check if an optional variable was passed to the macro is to use ``if(${ARGC} GREATER 2)``.
In the last case, you can use ``foreach(loop_var ${ARGN})`` but this will skip empty arguments.
If you need to include them, you can use 

.. code-block:: cmake

 set(list_var "${ARGN}")
 foreach(loop_var IN LISTS list_var)

Note that if you have a variable with the same name in the scope from which the macro is called, using unreferenced names will use the existing variable instead of the arguments.
For example:

.. code-block:: cmake

 macro(bar)
   foreach(arg IN LISTS ARGN)
     <commands>
   endforeach()
 endmacro()

 function(foo)
   bar(x y z)
 endfunction()

 foo(a b c)

Will loop over ``a;b;c`` and not over ``x;y;z`` as one might have expected.
If you want true CMake variables and/or better CMake scope control you should look at the function command.

参考情報
^^^^^^^^

* :command:`cmake_parse_arguments`
* :command:`endmacro`
