execute_process
---------------

一つ以上の子プロセスを実行する。

.. code-block:: cmake

  execute_process(COMMAND <cmd1> [<arguments>]
                  [COMMAND <cmd2> [<arguments>]]...
                  [WORKING_DIRECTORY <directory>]
                  [TIMEOUT <seconds>]
                  [RESULT_VARIABLE <variable>]
                  [RESULTS_VARIABLE <variable>]
                  [OUTPUT_VARIABLE <variable>]
                  [ERROR_VARIABLE <variable>]
                  [INPUT_FILE <file>]
                  [OUTPUT_FILE <file>]
                  [ERROR_FILE <file>]
                  [OUTPUT_QUIET]
                  [ERROR_QUIET]
                  [COMMAND_ECHO <where>]
                  [OUTPUT_STRIP_TRAILING_WHITESPACE]
                  [ERROR_STRIP_TRAILING_WHITESPACE]
                  [ENCODING <name>]
                  [ECHO_OUTPUT_VARIABLE]
                  [ECHO_ERROR_VARIABLE]
                  [COMMAND_ERROR_IS_FATAL <ANY|LAST>])

指定した一つ以上のコマンドからなるシーケンスを実行します。

これらのコマンドはパイプラインとして実行されていき、各プロセスの標準出力が、次のプロセスの標準入力にパイプされます。
一個の標準エラー出力へのパイプが全てのプロセス間で共有されます。

この ``execute_process`` コマンドは、CMake がビルドシステムを生成する前、すなわちプロジェクトの構成中に実行されます。
さらに :command:`add_custom_target` や :command:`add_custom_command` コマンドを使って、ビルド時に実行するカスタムコマンドを生成できます。

利用できるオプションは次のとおりです：

``COMMAND``
 子プロセスで実行するコマンドラインを指定する。

 CMake はオペレーティング・システムの API を直接使用して子プロセスを実行する。

 * POSIX 系プラットフォームの場合、コマンドラインは ``argv[]`` 系の配列で子プロセスに渡される。

 * Windows 系プラットフォームの場合、コマンドラインは文字列としてエンコードされ、``CommandLineToArgvW()`` 関数を使う子プロセスがそれをデコードする。

 パイプラインの途中でシェルは使われないので、``>`` のようなシェルの演算子は通常の文字として扱う（そのため、標準入力や標準出力、あるいは標準エラー出力を子プロセスにリダイレクトする場合は、それぞれ ``INPUT_*``、``OUTPUT_*``、そして ``ERROR_*`` を使うこと）。

 複数のコマンドを **順番に実行する** には、それぞれの ``COMMAND`` を引数として渡した ``execute_process`` を複数回呼び出すこと。

``WORKING_DIRECTORY``
 子プロセスの作業ディレクトリを指定する。

``TIMEOUT``
 この後ろに指定した秒数が経過すると、まだ完了していない子プロセスを全て強制終了し、変数の ``RESULT_VARIABLE`` に "timeout" と云う文字列を格納する。

``RESULT_VARIABLE``
 最後の子プロセスの実行結果を格納する変数を指定する。
 この変数には、最後の子プロセスからの返り値コード（整数値）か、またはエラーの状態を説明する文字列が格納される。

``RESULTS_VARIABLE <variable>``
 .. versionadded:: 3.10

 指定された ``COMMAND`` の順番でセミコロンで区切った子プロセスの結果を要素とする :ref:`リスト <CMake Language Lists>` を ``<variable>`` に格納する。
 このリストの要素は、子プロセスからの返り値コード（整数値）か、またはエラーの状態を説明する文字列である。

``INPUT_FILE <file>``
  「*最初の*」 ``COMMAND`` を実行する子プロセスの標準入力に接続するパイプ ``<file>`` を指定する。

``OUTPUT_FILE <file>``
  「*最後の」* ``COMMAND`` を実行する子プロセスの標準出力に接続するパイプ ``<file>`` を指定する。

``ERROR_FILE <file>``
  「*全ての」* ``COMMAND`` を実行する子プロセスの標準エラー出力に接続するパイプ ``<file>`` を指定する。

.. versionadded:: 3.3
  この ``<file>`` と同じ名前のパイプが ``OUTPUT_FILE`` と ``ERROR_FILE`` に指定された場合は、その ``<file>`` が標準出力と標準エラー出力の両方のパイプとして使用できるようになった。

``OUTPUT_QUIET``, ``ERROR_QUIET``
 ``OUTPUT_VARIABLE`` の標準出力、または ``ERROR_VARIABLE`` の標準エラー出力に接続しない（変数は空のまま）。
 ``*_FILE`` と ``ECHO_*_VARIABLE`` 類のオプションには影響しない。

``OUTPUT_VARIABLE``, ``ERROR_VARIABLE``
 指定した変数には、標準出力と標準エラー出力のパイプの内容が格納される。
 双方に同じ名前の変数を指定すると、それぞれの出力を生成された時間順に結合して格納する。

``ECHO_OUTPUT_VARIABLE``, ``ECHO_ERROR_VARIABLE``
  .. versionadded:: 3.18

  標準出力または標準エラー出力を、排他的に変数にはリダイレクトしない。

  コマンドの出力は、指定した変数にコピーされ、UNIX 系の ``tee`` コマンドに似た方法で標準出力または標準エラー出力にもリダイレクトされる。

.. note::
  複数の ``OUTPUT_*`` や ``ERROR_*`` 類のオプションに、同じパイプを指定した場合、優先順位を「*指定することはできません*」。
  また ``OUTPUT_*`` や ``ERROR_*`` 類のオプションを指定しない場合、CMake コマンドを実行したプロセスが接続しているパイプをそれぞれ使用します。

``COMMAND_ECHO <where>``
 .. versionadded:: 3.15

 実行中のコマンド出力を ``<where>`` にエコーする（``<where>`` は ``STDERR``、``STDOUT``、``NONE`` のいずれか）。
 このオプションを指定しない場合のデフォルトの挙動を制御する方法について、CMake 変数の :variable:`CMAKE_EXECUTE_PROCESS_COMMAND_ECHO` を参照のこと。

``ENCODING <name>``
 .. versionadded:: 3.8

 Windows 系プラットフォームの場合に、プロセスから受け取った出力をデコードする際に使用するエンコーディングを指定する。
 このオプションは他のプラットフォームでは無視される。
 指定できるエンコーディングは次のとおり：

 ``NONE``
   デコードは行わない。そのため、プロセスの出力は CMake が内部で使用するエンコーディング（UTF-8）と同じ方法でエンコードされているものと仮定する。
   これがデフォルト。
 ``AUTO``
   現在使用しているコンソールのコードページか、またはそれが利用できない場合は ANSI のコードページを使う。
 ``ANSI``
   ANSI のコードページを使う。
 ``OEM``
   OEM（相手先商標製品製造業者）指定のコードページを使う。
 ``UTF8`` or ``UTF-8``
   UTF-8 のコードページを使う。

   .. versionadded:: 3.11
    `UTF-8 RFC <https://www.ietf.org/rfc/rfc3629>`_ の命名規則との一貫性を保つために ``UTF-8`` エンコーディングの文字列を受け取れるようになった。

``COMMAND_ERROR_IS_FATAL <ANY|LAST>``
  .. versionadded:: 3.19

  この ``COMMAND_ERROR_IS_FATAL`` に続く次のオプションで、エラーが発生した時の挙動を決定する：

    ``ANY``
      リストに含まれるコマンドのいずれか一つが失敗したら、``execute_process()`` コマンドはエラーで停止する。

    ``LAST``
      リストに含まれるコマンドの最後のコマンドが失敗したら、``execute_process()`` コマンドはエラーで停止する。
