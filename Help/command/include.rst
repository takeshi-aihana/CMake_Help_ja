include
-------

ファイルやモジュールから CMake のコードを読み込んで実行する。

.. code-block:: cmake

  include(<file|module> [OPTIONAL] [RESULT_VARIABLE <var>]
                        [NO_POLICY_SCOPE])

``<file>`` や ``<module>`` から CMake のコードを読み込んで実行します。
このコマンドを呼び出した時のスコープ内で、``<var>`` などの変数の読み込みや書き込みを行えます（動的スコープ方式）。
``OPTIONAL`` を指定すると、``<file>`` や ``<moduleis>`` が見つからなくてもエラーにはなりません。
``RESULT_VARIABLE`` を指定すると、引数の ``<var>`` は読み込んだファイルのフルパス名がセットされるか、あるいはファイルの読み込みに失敗したら ``NOTFOUND`` がセットされます。

ファイルの代わりにモジュールを渡すと、モジュール・ファイル（``<modulename>.cmake``）を、まず :variable:`CMAKE_MODULE_PATH` から検索し、次に CMake のモジュール・ディレクトリから検索します。
これには例外が一つあります：
``include()`` コマンドを呼び出す CMake ファイル自身がモジュール・ディレクトリにある場合、最初にモジュール・ディレクトリを検索し、その後に :variable:`CMAKE_MODULE_PATH` を検索します。
:policy:`CMP0017` ポリシーも参照して下さい。

``NO_POLICY_SCOPE`` オプションの使い方など詳細は :command:`cmake_policy` コマンドを参照して下さい。
