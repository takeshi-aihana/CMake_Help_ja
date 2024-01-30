site_name
---------

コンピュータの名前をセットする。

.. code-block:: cmake

  site_name(variable)

``variable`` の値をホスト名にセットします。

なお CMake を実行するホストが UNIX 系のプラットフォームで、環境変数の ``HOSTNAME`` が既に定義されている場合は、``hostname`` コマンドライン・ツールと同様に、ホスト名を出力するだけです。
