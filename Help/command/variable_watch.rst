variable_watch
--------------

CMake 変数へのアクセスを監視する。

.. code-block:: cmake

  variable_watch(<variable> [<command>])

``<command>`` が指定されていない時に、``<variable>`` がアクセス（読み込みまたは書き込み）されたら、その旨を知らせるメッセージを出力します。

``<command>`` を指定すると、メッセージを出力する代わりに、その ``<command>`` を実行します。
``<command>`` は次に示す引数を受け取ります：
``COMMAND(<variable> <access> <value> <current_list_file> <stack>)``

``<variable>``
 監視している変数名。

``<access>``
 ``READ_ACCESS``、``UNKNOWN_READ_ACCESS``、``MODIFIED_ACCESS``、``UNKNOWN_MODIFIED_ACCESS``、または ``REMOVED_ACCESS`` のいずれか。
 ``UNKNOWN_`` 系は ``<variable>`` に一度も値がセットされていない場合にのみ指定できる。
 これを指定したあと、``<variable>`` が :command:`unset` されたら同じ CMake の実行プロセス中は監視しなくなる。

``<value>``
 ``<variable>`` の値。
 ``<variable>`` が変更されたら、この値が ``<variable>`` の新しい値になる。
 ``<variable>`` が削除されたら、この値は空である。

``<current_list_file>``
 変数を参照したファイルの絶対パス。

``<stack>``
 現在スタックにプッシュされている全てのファイルの絶対パスを要素した :ref:`リスト <CMake Language Lists>`。
 このリストの先頭の要素は最下位のファイルで、最後の要素は現在処理中のファイル（すなわち ``current_list_file``）である。

:command:`list(APPEND)` コマンドような一部の処理では、この ``variable_watch`` コマンドが二回呼び出されるので注意して下さい（一回目は読み取りの監視、二回目は書き込みの監視です）。
また、``<variable>`` に対して :command:`if(DEFINED)` コマンドを実行しても、アクセスされたとはみなされず、この ``variable_watch`` コマンドが実行されない点にも注意して下さい。

このコマンドで監視できる変数は、通常の変数だけです。
キャッシュ変数のアクセスは監視しません。
なお、キャッシュ変数として ``var`` が存在し、通常変数の ``var`` が存在していない場合に、通常変数の ``var`` への ``<access>`` は ``UNKNOWN_`` 系にはならないので注意して下さい。
