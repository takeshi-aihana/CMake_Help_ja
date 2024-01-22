include_guard
-------------

.. versionadded:: 3.10

CMake が現在処理している CMake ファイルにインクルード・ガードの機能を提供する。

.. code-block:: cmake

  include_guard([DIRECTORY|GLOBAL])

現在の CMake ファイルのインクルード・ガード [#include_guard]_ 機能を設定します（CMake ファイルについては :variable:`CMAKE_CURRENT_LIST_FILE` 変数も参照して下さい）。

該当するスコープ内で CMake ファイルが既に処理されている場合、``include_gurad`` コマンドを呼び出した時点で処理を終了します。
これにより、ヘッダファイルで一般的に使用されるインクルード・ガードや ``#pragma once`` 命令と同等の機能を実現します。
これは、該当するスコープ内で、以前にこの CMake ファイルが処理されていることを検出したら :command:`return` コマンドを呼び出します。
ただし、:command:`function` の中で ``include_guard`` コマンドを呼び出さないで下さい。

ガードするスコープをオプションとして指定できます。
指定できる値は次のとおりです：

``DIRECTORY``
  インクルード・ガードの適用を、現在のディレクトリ以下に限定する。
  現在のディレクトリにあるファイルは一回だけ読み込めるが、それ以外のディレクトリ [#option_DIRECTORY]_ にあるファイルは再び読み込みことは可能。

``GLOBAL``
  インクルード・ガードをビルド全体（グローバル）に適用する。
  現在のファイルは、スコープにかかわらず一回だけ読み込まれる。

オプションを指定しなかった場合、``include_guard`` は任意の変数と同じスコープを持ちます。つまりインクルード・ガードの効果は直近の関数スコープまたは現在のディレクトリのどちらかがが対象になります。
この場合、``include_guard`` コマンドの動作は次のコードと等価です：

.. code-block:: cmake

  if(__CURRENT_FILE_VAR__)
    return()
  endif()
  set(__CURRENT_FILE_VAR__ TRUE)

.. rubric:: Footnotes

.. [#include_guard] ソースコードのヘッダファイルを２回以上インクルードさせないためにプリプロセッサとして指示する機能。
.. [#option_DIRECTORY] 現在読み込んでいるファイルなどから :command:`add_subdirectory` や :command:`include` コマンドで追加したりインクルードしていない別のディレクトリ（例えば親ディレクトリ）。
