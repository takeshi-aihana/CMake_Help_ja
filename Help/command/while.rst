while
-----

条件が true の間、コマンドのグループを評価する。

.. code-block:: cmake

  while(<condition>)
    <commands>
  endwhile()

この ``while`` と対になる :command:`endwhile` の間にある ``<commands>`` は、呼び出されずに記録されます。
``endwhile`` コマンドが評価された後、``<condition>`` が ``TRUE`` である間、記録されていた全ての ``<commands>`` が呼び出されます。

``<condition>`` は、:command:`if` コマンドの場合と同じ「:ref:`Condition Syntax`」に従った式にして下さい。

:command:`break` と :command:`continue` コマンドは通常の制御フロー（*Control Flow*）から抜け出す手段を提供します。

従来どおり、:command:`endwhile` コマンドはオプションで ``<condition>`` を引数として受け取れます。
その場合、最初の ``while`` コマンドの引数をそのまま繰り返すようにして下さい。

参考情報
^^^^^^^^

* :command:`break`
* :command:`continue`
* :command:`foreach`
* :command:`endwhile`
