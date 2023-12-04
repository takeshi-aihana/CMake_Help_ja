continue
--------

.. versionadded:: 3.2

囲んでいるループ・ブロック（foreach や while コマンド）の先頭へジャンプして処理を継続する。

.. code-block:: cmake

  continue()

この ``continue()`` コマンドを使うと、CMake スクリプトで :command:`foreach` や :command:`while` で囲んでいる現在のブロックの残りの処理を中止して、ブロックの先頭から処理を開始します。

:command:`break` コマンドも参照して下さい。
