build_command
-------------

プロジェクトをビルドするためのコマンドラインを取得する（主に :module:`CTest` モジュール内で使用するコマンド）。

.. code-block:: cmake

  build_command(<variable>
                [CONFIGURATION <config>]
                [PARALLEL_LEVEL <parallel>]
                [TARGET <target>]
                [PROJECT_NAME <projname>] # legacy, causes warning
               )

次の書式のコマンドライン文字列 を ``<variable>`` にセットします::

 <cmake> --build . [--config <config>] [--parallel <parallel>] [--target <target>...] [-- -i]

ここで ``<cmake>`` は :manual:`cmake(1)` コマンドライン・ツールのパス名であり、``<config>`` と ``<parallel>`` と ``<target>`` は、それぞれオプションの ``CONFIGURATION`` と ``PARALLEL_LEVEL`` と ``TARGET`` に指定した値で置き換えられます。
:policy:`CMP0061` のポリシーが ``OLD``  の場合、コマンドライン末尾にある ``-- -i`` の部分が :ref:`Makefile Generators` に追加されます。

この ``build_command`` コマンドを呼び出すと、:option:`cmake --build` から始まるコマンドラインがビルドシステムのビルド・ツールを起動します。

.. versionadded:: 3.21
  ``PARALLEL_LEVEL`` オプションで :option:`--parallel <cmake--build --parallel>` フラグを指定できるようになった。

.. code-block:: cmake

  build_command(<cachevariable> <makecommand>)

この呼び出し方は推奨されていませんが、下位互換性を維持する目的でのみ利用可能です。
代わりに、冒頭に提示した呼び出し方を使って下さい。

コマンドライン文字列を ``<cachevariable>`` にセットしますが、この中には :option:`--target <cmake--build --target>` オプション列は含まれません。
通常 ``<makecommand>`` は無視されますが、``devenv`` や ``nmake`` や ``make``、または旧式のビルド・ツールを使っている場合は、そのツールの絶対パスを指定して下さい。

.. note::
 バージョン 3.0 より古い CMake の場合、このコマンドは使用するジェネレータに対応するネィティブなビルド・ツールを直接呼び出すコマンドラインを返していた。
 ``PROJECT_NAME`` オプションがあまり効果がなかったので、このオプションを指定すると CMake は警告するようになった。
