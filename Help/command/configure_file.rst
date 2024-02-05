configure_file
--------------

ファイルを別の場所にコピーして、その内容を変更する。

.. code-block:: cmake

  configure_file(<input> <output>
                 [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS |
                  FILE_PERMISSIONS <permissions>...]
                 [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
                 [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])

``<input>`` ファイルを ``<output>`` ファイルにコピーし、そのファイルの中にある ``@VAR@``、``${VAR}``、``$CACHE{VAR}``、``$ENV{VAR}`` として参照できる変数の値を置き換えます。
変数の参照部分はそれぞれ、その変数が現在保持している値で置き換えられるか、またはその変数が定義されていなかったら空の文字列で置き換えられます。
さらに、次のような書式を持つ行は

.. code-block:: c

  #cmakedefine VAR ...

次のいずれかで置き換えられます：

.. code-block:: c

  #define VAR ...

または

.. code-block:: c

  /* #undef VAR */

どちらで置き換えられるかは ``VAR`` という変数に何がセットされているかに依存します（たとえば :command:`if` コマンドで ``VAR`` を判定した結果）。
また ``VAR`` につづく "``...``" の部分に何か指定されている場合は、``VAR`` が "``...``" の文字列で置き換えられます。
さらに ``#cmakedefine01 VAR`` という書式を持つ行は ``VAR`` 自身が ``VAR 0`` または ``VAR 1`` に展開されます。
したがって、次のような書式を持つ行は

.. code-block:: c

  #cmakedefine01 VAR

次のいずれかで置き換えられます：

.. code-block:: c

  #define VAR 0

または

.. code-block:: c

  #define VAR 1

ここで ``#cmakedefine01 VAR ...`` という書式にしてしまうと ``#cmakedefine01 VAR ... 0`` または ``#cmakedefine01 VAR ... 1`` に展開されてしまい、未定義の動作になる可能性があります。

.. versionadded:: 3.10
  行頭におく ``#`` 文字と ``cmakedefine`` または ``cmakedefine01`` キーワードの間に空白やタブ文字を挿入してインデントさせることができるようになりました（ただし ``#undef`` キーワードは除く）。
  この空白を利用したインデントは変換後も維持されます：

  .. code-block:: c

    #  cmakedefine VAR
    #  cmakedefine01 VAR

  これは ``VAR`` が定義されていたら、次のように置き換えられます：

  .. code-block:: c

    #  define VAR
    #  define VAR 1

``<input>`` ファイルが変更されると、ビルドシステムは CMake を再実行してファイルを再構成し、ビルドシステムを再度生成します。
ここで作成されたファイルは、その内容が変更された場合にだけ更新されます。

このコマンドに指定できるオプションは次のとおりです：

``<input>``
  コピー元のファイルのパス名。
  相対パスを指定すると、:variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとして絶対パスを計算する。
  このパスはディレクトリではなくファイルにすること。

``<output>``
  コピー先のファイルまたはディレクトリへのパス名。
  相対パスを指定すると、:variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリとして絶対パスを計算する。
  既に存在しているディレクトリを指定した場合、出力先は入力ファイルと同じ名前のファイルで、指定したディレクトリ下にコピーされる。
  パスに存在していないディレクトリが含まれている場合は、まずそのディレクトリを作成してからコピーする。

``NO_SOURCE_PERMISSIONS``
  .. versionadded:: 3.19

  コピー元のアクセス権限をコピー先のファイルに適用しない。
  コピー先のアクセス権限は、デフォルトで標準の 644 (``-rw-r--r--``) が適用される。

``USE_SOURCE_PERMISSIONS``
  .. versionadded:: 3.20

  コピー元のアクセス権限をコピー先のファイルに適用する。
  アクセス権限に関連する3つのオプション（``NO_SOURCE_PERMISSIONS`` と ``USE_SOURCE_PERMISSIONS`` と ``FILE_PERMISSIONS``）がいずれも指定されていない場合は、この対応がデフォルトである。
  この ``USE_SOURCE_PERMISSIONS`` オプションは主に、コマンドの呼び出し側で意図した対応を明示的に実現する方法である。

``FILE_PERMISSIONS <permissions>...``
  .. versionadded:: 3.20

  コピー元のアクセス権限を無視して、代わりに指定した ``<permissions>`` をコピー先に適用する。

``COPYONLY``
  変数の値を置き換えたり、その他の内容を書き換えることはせずに、単にファイルをコピーするだけ。
  これは ``NEWLINE_STYLE`` オプションと一緒には指定できない。

``ESCAPE_QUOTES``
  置き換えたあとにクォート文字をバックスラッシュでエスケープする（C言語方式）。

``@ONLY``
  変数の値の置き換えを ``@VAR@`` だけに制限する。
  これは ``${VAR}`` を使うスクリプトを構成する際に便利である。

``NEWLINE_STYLE <style>``
  コピー先の改行スタイルを指定する。
  指定可能なスタイルは、改行文字が ``\n`` の場合は ``UNIX`` または ``LF``、 改行文字が ``\r\n`` の場合は ``DOS``、``WIN32`` または ``CRLF`` である。
  これは ``COPYONLY`` オプションと一緒には指定できない。

例
^^

以下の内容を持った ``foo.h.in`` というファイルがソースツリーにある場合を考えてみます：

.. code-block:: c

  #cmakedefine FOO_ENABLE
  #cmakedefine FOO_STRING "@FOO_STRING@"

``CMakeLists.txt`` では ``configure_file`` コマンドを使ってヘッダ・ファイルを作成します：

.. code-block:: cmake

  option(FOO_ENABLE "Enable Foo" ON)
  if(FOO_ENABLE)
    set(FOO_STRING "foo")
  endif()
  configure_file(foo.h.in foo.h @ONLY)

これにより、ソースツリーに対応するビルドツリーに ``foo.h`` というヘッダ・ファイルが作成されます。
``FOO_ENABLE`` が ``ON`` の場合は以下の内容に変換されます：

.. code-block:: c

  #define FOO_ENABLE
  #define FOO_STRING "foo"

それ以外の場合は以下の内容に変換されます：

.. code-block:: c

  /* #undef FOO_ENABLE */
  /* #undef FOO_STRING */

次に :command:`target_include_directories` コマンドを使って、ヘッダ・ファイルが作成されたディレクトリをインクルード・ディレクトリとして指定します：

.. code-block:: cmake

  target_include_directories(<target> [SYSTEM] <INTERFACE|PUBLIC|PRIVATE> "${CMAKE_CURRENT_BINARY_DIR}")

その結果、ソース・ファイルでは ``#include <foo.h>`` のようにヘッダ・ファイルをインクルードすることができます。

参考情報
^^^^^^^^

* :command:`file(GENERATE)`
