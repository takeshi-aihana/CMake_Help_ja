endforeach
----------

foreach のブロックの中で実行するコマンド列を終了する。

.. code-block:: cmake

  endforeach([<loop_var>])

:command:`foreach` コマンドを参照して下さい。

The optional ``<loop_var>`` argument is supported for backward compatibility only.
If used it must be a verbatim repeat of the ``<loop_var>`` argument of the opening ``foreach`` clause.
