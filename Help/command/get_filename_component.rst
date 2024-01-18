get_filename_component
----------------------

完全なファイル名から特定の要素を取得する。

.. versionchanged:: 3.20
  このコマンドは :command:`cmake_path` コマンドに置き換えられた（ただし例外として ``REALPATH`` は :command:`file(REAL_PATH)` を使い、``PROGRAM`` は :command:`separate_arguments(PROGRAM)` を使う）。


.. versionchanged:: 3.24
  Windows レジストリをクエリする機能は :ref:`cmake_host_system_information(QUERY WINDOWS_REGISTRY)<Query Windows registry>` コマンドで置き換えられた。

.. code-block:: cmake

  get_filename_component(<var> <FileName> <mode> [CACHE])

``<FileName>`` の任意の要素を取得して ``<var>`` に格納します。取得する要素を表す ``<mode>`` には、次のいずれかを指定して下さい：

::

 DIRECTORY = ファイル名を含めないディレクトリ
 NAME      = ディレクトリを含めないファイル名
 EXT       = 最長の拡張子（たとえば d/a.b.c ならば .b.c）
 NAME_WE   = ディレクトリや最長の拡張子を除いた部分
 LAST_EXT  = 最短の拡張子（たとえば d/a.b.c ならば .c）
 NAME_WLE  = ディレクトリや最短の拡張子を除いた部分
 PATH      = ディレクトリ部分（CMake <= バージョン 2.8.11 向け）

.. versionadded:: 3.14
  要素として ``LAST_EXT`` と ``NAME_WLE`` を追加した。

取得した要素の先頭には "``/``" （スラッシュ）が付いてますが、末尾に "``/``" （スラッシュ）は付きません。
``CACHE`` オプションを指定すると、``<var>`` はキャッシュ変数になります。

.. code-block:: cmake

  get_filename_component(<var> <FileName> <mode> [BASE_DIR <dir>] [CACHE])

.. versionadded:: 3.4

``<FileName>`` からパスを取得して ``<var>`` に格納します。``<mode>`` には、次のいずれかを指定して下さい：

::

 ABSOLUTE  = ファイルの絶対パス
 REALPATH  = シンボリックリンクが解決されたファイルの絶対パス

``<FileName>`` が相対パスの場合、``<dir>`` を起点としたパスとして評価します。
``<dir>`` を指定しない場合、デフォルトの起点は :variable:`CMAKE_CURRENT_SOURCE_DIR` です。

取得したパスの先頭には "``/``" （スラッシュ）が付いてますが、末尾に "``/``" （スラッシュ）は付きません。
``CACHE`` オプションを指定すると、``<var>`` はキャッシュ変数になります。

.. code-block:: cmake

  get_filename_component(<var> <FileName> PROGRAM [PROGRAM_ARGS <arg_var>] [CACHE])

``<FileName>`` に含まれているプログラム名はシステムの検索パスまたは絶対パスのままです。
``PROGRAM_ARGS`` が ``PROGRAM`` と一緒に存在している場合、``<FileName>`` の中にあるコマンドライン引数はプログラムから分離され、``<arg_var>`` 変数に格納されます。
これによりプログラム名とその引数をそれぞれ別個に参照できます。

参考情報
^^^^^^^^

* :command:`cmake_path`
