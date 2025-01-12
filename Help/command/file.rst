file
----

ファイルを操作するコマンド。

このコマンドは、ファイルやパスを操作するためにファイルシステムへのアクセスを必要とする。

これに対して、CMake 上でファイルやパスを操作するコマンドについては :command:`cmake_path` コマンドを参照のこと。

.. note::

  このコマンドのサブコマンド `RELATIVE_PATH`_、 `TO_CMAKE_PATH`_ 、そして `TO_NATIVE_PATH`_ は、それぞれ :command:`cmake_path` コマンドのサブコマンド :ref:`RELATIVE_PATH <cmake_path-RELATIVE_PATH>`、:ref:`CONVERT ... TO_CMAKE_PATH_LIST <cmake_path-TO_CMAKE_PATH_LIST>` 、そして :ref:`CONVERT ... TO_NATIVE_PATH_LIST <cmake_path-TO_NATIVE_PATH_LIST>` に置き換えられました。

概要
^^^^

.. parsed-literal::

  `ファイルを読み込む`_
    file(`READ`_ <filename> <out-var> [...])
    file(`STRINGS`_ <filename> <out-var> [...])
    file(`\<HASH\>`_ <filename> <out-var>)
    file(`TIMESTAMP`_ <filename> <out-var> [...])
    file(`GET_RUNTIME_DEPENDENCIES`_ [...])

  `ファイルに書き込む`_
    file({`WRITE`_ | `APPEND`_} <filename> <content>...)
    file({`TOUCH`_ | `TOUCH_NOCREATE`_} [<file>...])
    file(`GENERATE`_ OUTPUT <output-file> [...])
    file(`CONFIGURE`_ OUTPUT <output-file> CONTENT <content> [...])

  `ファイルシステムを使う`_
    file({`GLOB`_ | `GLOB_RECURSE`_} <out-var> [...] [<globbing-expr>...])
    file(`MAKE_DIRECTORY`_ [<dir>...])
    file({`REMOVE`_ | `REMOVE_RECURSE`_ } [<files>...])
    file(`RENAME`_ <oldname> <newname> [...])
    file(`COPY_FILE`_ <oldname> <newname> [...])
    file({`COPY`_ | `INSTALL`_} <file>... DESTINATION <dir> [...])
    file(`SIZE`_ <filename> <out-var>)
    file(`READ_SYMLINK`_ <linkname> <out-var>)
    file(`CREATE_LINK`_ <original> <linkname> [...])
    file(`CHMOD`_ <files>... <directories>... PERMISSIONS <permissions>... [...])
    file(`CHMOD_RECURSE`_ <files>... <directories>... PERMISSIONS <permissions>... [...])

  `パスを評価する`_
    file(`REAL_PATH`_ <path> <out-var> [BASE_DIRECTORY <dir>] [EXPAND_TILDE])
    file(`RELATIVE_PATH`_ <out-var> <directory> <file>)
    file({`TO_CMAKE_PATH`_ | `TO_NATIVE_PATH`_} <path> <out-var>)

  `ファイルを転送する`_
    file(`DOWNLOAD`_ <url> [<file>] [...])
    file(`UPLOAD`_ <file> <url> [...])

  `ファイルをロックする`_
    file(`LOCK`_ <path> [...])

  `アーカイブを扱う`_
    file(`ARCHIVE_CREATE`_ OUTPUT <archive> PATHS <paths>... [...])
    file(`ARCHIVE_EXTRACT`_ INPUT <archive> [...])

ファイルを読み込む
^^^^^^^^^^^^^^^^^^

.. signature::
  file(READ <filename> <variable>
       [OFFSET <offset>] [LIMIT <max-in>] [HEX])

  ``<filename>`` というファイルを読み込んで、その内容を ``<variable>`` に格納します。
  オプションで、指定した位置 ``<offset>`` から読み込みを開始し、最大で ``<max-in>`` バイトを読み取ることもできます。
  ``HEX`` オプションを指定すると、読み込んだデータを16進数表記に変換します（これはバイナリデータの読み込みに便利です）。
  この ``HEX`` オプションを指定すると、出力する文字（``a`` から ``f``）は小文字になります。

.. signature::
  file(STRINGS <filename> <variable> [<options>...])

  ``<filename>`` というファイルを読み込んで 一行分の ASCII 文字列を要素とするリストを変換し、それを ``<variable>`` に格納します。
  ファイルにあるバイナリデータは無視します。
  キャリッジリターン文字（``\r`` や CR）は無視します。
  指定できるオプションは次のとおりです:

    ``LENGTH_MAXIMUM <max-len>``
      最大で ``<max-len>`` の長さの文字列だけ解析する。

    ``LENGTH_MINIMUM <min-len>``
      最低で ``<min-len>`` の長さの文字列だけ解析する。

    ``LIMIT_COUNT <max-num>``
      最大で ``<max-num>`` 個の文字列（個別）を読み込む。

    ``LIMIT_INPUT <max-in>``
      ファイルから読み込むバイト数を ``<min-num>`` にする。

    ``LIMIT_OUTPUT <max-out>``
      ``<variable>`` に格納するバイト数の合計を ``<max-out>`` にする。

    ``NEWLINE_CONSUME``
      改行文字（``\n`` や LF）を文字列の一部として扱う。

    ``NO_HEX_CONVERSION``
      このオプションを指定すると、Intel Hex と Motorola S-レコードのファイルの場合、自動的にバイナリデータには変換しない。

    ``REGEX <regex>``
      正規表現の ``<regex>`` にマッチする文字列だけ読み込む。正規表現については :ref:`string(REGEX) <Regex Specification>` を参照のこと。

    ``ENCODING <encoding-type>``
      .. versionadded:: 3.1

      読み込んだ文字列を ``<encoding-type>`` のエンコーディングで扱う。現在サポートしているエンコーディングは、``UTF-8``、``UTF-16LE``、``UTF-16BE``、``UTF-32LE``、``UTF-32BE`` 。
      ``ENCODING`` オプションを指定せず、ファイルにバイト・オーダーのマークがある場合、``ENCODING`` オプションはバイト・オーダー・マークをデフォルトで尊重する。

  .. versionadded:: 3.2
    ``UTF-16LE``、``UTF-16BE``、``UTF-32LE``、そして ``UTF-32BE`` のエンコーディングが追加された。

  たとえば、次のコマンドは：

  .. code-block:: cmake

    file(STRINGS myfile.txt myfile)

  ファイル ``myfile.txt`` を読み込んで、各行を要素とするリストを作成し、それを変数の ``myfile`` に格納します。

.. signature::
  file(<HASH> <filename> <variable>)
  :target: <HASH>

  ``<filename>`` の内容に対するハッシュを計算し、それを ``<variable>`` に格納します。
  サポートしている ``<HASH>`` アルゴリズムの名前はe :command:`string(<HASH>)` コマンドを参照して下さい。

.. signature::
  file(TIMESTAMP <filename> <variable> [<format>] [UTC])

  ``<filename>`` のタイムスタンプから変更時刻を表す文字列を作成し、それを ``<variable>`` に格納します。
  タイムスタンプを取得できない場合は、空文字（""）を格納します。

  指定できる ``<format>`` や ``UTFC`` オプションについては :command:`string(TIMESTAMP)` コマンドを参照してく下さい。

.. signature::
  file(GET_RUNTIME_DEPENDENCIES [...])

  .. versionadded:: 3.16

  指定したファイル（インストールするファイル）が依存しているファイル（ライブラリ）を要素とするリストを再帰的に取得します：

  .. code-block:: cmake

    file(GET_RUNTIME_DEPENDENCIES
      [RESOLVED_DEPENDENCIES_VAR <deps_var>]
      [UNRESOLVED_DEPENDENCIES_VAR <unresolved_deps_var>]
      [CONFLICTING_DEPENDENCIES_PREFIX <conflicting_deps_prefix>]
      [EXECUTABLES [<executable_files>...]]
      [LIBRARIES [<library_files>...]]
      [MODULES [<module_files>...]]
      [DIRECTORIES [<directories>...]]
      [BUNDLE_EXECUTABLE <bundle_executable_file>]
      [PRE_INCLUDE_REGEXES [<regexes>...]]
      [PRE_EXCLUDE_REGEXES [<regexes>...]]
      [POST_INCLUDE_REGEXES [<regexes>...]]
      [POST_EXCLUDE_REGEXES [<regexes>...]]
      [POST_INCLUDE_FILES [<files>...]]
      [POST_EXCLUDE_FILES [<files>...]]
      )

  これらのサブコマンドは CMake プロジェクトの構成中に使うことを意図したものではない点に注意して下さい。
  すなわち :command:`install(RUNTIME_DEPENDENCY_SET)` コマンドで生成したコード、または :command:`install(CODE)` や :command:`install(SCRIPT)` を介してプロジェクトから提供されたコードで、ファイルをインストールする際に使用することを意図しています。
  たとえば、次のように使います：

  .. code-block:: cmake

    install(CODE [[
      file(GET_RUNTIME_DEPENDENCIES
        # ...
        )
      ]])

  このコマンドに指定できる引数は次のとおりです：

    ``RESOLVED_DEPENDENCIES_VAR <deps_var>``
      解決できた依存関係のリストを格納する変数を指定する。

    ``UNRESOLVED_DEPENDENCIES_VAR <unresolved_deps_var>``
      解決できなかった依存関係のリストを格納する変数を指定する。
      この変数を指定しない場合に、依存関係を解決できなかったらエラーを発行する。

    ``CONFLICTING_DEPENDENCIES_PREFIX <conflicting_deps_prefix>``
      競合する依存関係の情報を格納する変数の接頭詞を指定する。
      CMake では、同じ名前を持つ二つのファイルが別々のディレクトリに存在している場合を、依存関係が競合しているという。
      競合するファイル名のリストが、ここで指定した接頭詞を持つ ``<conflicting_deps_prefix>_FILENAMES`` に格納される。
      また、競合するファイルが見つかったパス名のリストも同様に ``<conflicting_deps_prefix>_<filename>`` に格納される。

    ``EXECUTABLES <executable_files>``
      依存関係を調べる実行形式のファイル名を :ref:`リスト <CMake Language Lists>` 形式で指定する。
      このリストは、通常は :command:`add_executable` コマンドで生成するものであるが、必ずしも CMake に作成させる必要はない。
      ホストが Apple 系のプラットフォームの場合、ライブラリの依存関係を再帰的に解決する時に、このリストを使って ``@executable_path`` の値を決定する。
      このリストにライブラリ（``STATIC``、``MODULE``、または ``SHARED``）を指定した場合の結果は未定義である。

    ``LIBRARIES <library_files>``
      依存関係を調べるライブラリのファイル名を :ref:`リスト <CMake Language Lists>` 形式で指定する。
      このリストは、通常は :command:`add_library(SHARED)` コマンドで生成するものであるが、必ずしも CMake に作成させる必要はない。
      このリストに ``STATIC`` ライブラリや ``MODULE`` 型のファイル、または実行形式を指定した場合の結果は未定義である。

    ``MODULES <module_files>``
      依存関係を調べるモジュール型のファイル名を :ref:`リスト <CMake Language Lists>` 形式で指定する。
      このリストは、通常は :command:`add_library(MODULE)` コマンドで生成するものであるが、必ずしも CMake に作成させる必要はない。
      この型のファイルは、リンク時に ``ld -l`` を使用してリンクされるものではなく、実行時に ``dlopen()`` を呼び出して使われる。
      このリストに ``STATIC`` ライブラリや ``SHARED`` ライブラリ、または実行形式を指定した場合の結果は未定義である。

    ``DIRECTORIES <directories>``
      依存関係を調べる際の追加ディレクトリを :ref:`リスト <CMake Language Lists>` 形式で指定する。
      ホストが Linux 系のプラットフォームの場合、標準の検索パスから依存関係が見つからなかった場合に、これらのディレクトリを追加で検索する。
      この追加ディレクトリから依存関係が見つからなかったら警告を発行する。これは、依存関係を調べるファイルのリンクが不完全なものであると判断するため（依存関係を含む全てのパスがリストされていない）。
      ホストが Windows 系のプラットフォームの場合、他の検索パスから依存関係が見つからなかった場合に、これらのディレクトリを追加で検索する（ただし、他の検索パスは Windows の依存関係の解決で基本となるディレクトリなので、見つからなくても警告は発行しない）。
      ホストが Apple 系のプラットフォームの場合、この引数は無視される。

    ``BUNDLE_EXECUTABLE <bundle_executable_file>``
      依存関係を解決する際に「バンドル実行形式（*Bundle Executable*） [#hint_for_framework_and_bundle_of_ios]_ 」として扱う実行形式を指定する。
      ホストが Apple 系プラットフォームの場合、 ``LIBRARIES`` と ``MODULES`` 型のファイルの依存関係を再帰的に解決する際に ``@executable_path`` を決定する。
      この引数は ``EXECUTABLES`` 型のファイルの場合は何もしない。
      ホストがそれ以外のプラットフォームの場合、この引数は何もしない。
      この引数は、通常は（ただし常にではないが） ``EXECUTABLES`` にリストされた実行形式のいずれかになる（パッケージの "main" 実行部）。

  次の引数で、任意のライブラリを依存関係の調査対象に含めるか含めないかを表すフィルタを指定できます。
  フィルタの仕組みについて詳細は、以下の説明を参照して下さい。

    ``PRE_INCLUDE_REGEXES <regexes>``
      まだ解決していない依存関係（ライブラリの名前）を調査対象に含める際に使用する pre-include 型の正規表現のリストを指定する。

    ``PRE_EXCLUDE_REGEXES <regexes>``
      まだ解決していない依存関係（ライブラリの名前）を調査対象から外す際に使用する pre-exclude 型の正規表現のリストを指定する。

    ``POST_INCLUDE_REGEXES <regexes>``
      解決した依存関係（ライブラリの名前）を調査対象に含める際に使用する post-include 型の正規表現のリストを指定する。

    ``POST_EXCLUDE_REGEXES <regexes>``
      解決した依存関係（ライブラリの名前）を調査対象から外す際に使用する post-exclude 型の正規表現のリストを指定する。

    ``POST_INCLUDE_FILES <files>``
      .. versionadded:: 3.21

      解決した依存関係（ライブラリの名前）を調査対象に含める際に使用する post-include 型のファイル名のリストを指定する。
      これらのファイル名にマッチするかどうかを確認する際に、シンボリックリンクを解決できる。

    ``POST_EXCLUDE_FILES <files>``
      .. versionadded:: 3.21

      解決した依存関係（ライブラリの名前）を調査対象から外す際に使用する post-exclude 型のファイル名のリストを指定する。
      これらのファイル名にマッチするかどうかを確認する際に、シンボリックリンクを解決できる。

  これらの引数を使って、依存関係を解決する時に不要なシステム・ライブラリを除外したり、特定のディレクトリにあるライブラリを依存関係に含めることができます。
  このフィルタは次のステップに従って機能します：

  1. まだ解決していない依存関係（ライブラリ）が ``PRE_INCLUDE_REGEXES`` のいずれかの正規表現にマッチする場合、ステップ 2 と 3 をスキップし、依存関係の解決はステップ 4 へ。

  2. まだ解決していない依存関係（ライブラリ）が ``PRE_EXCLUDE_REGEXES`` のいずれかの正規表現にマッチする場合、その依存関係の解決を停止する。

  3. それ以外は、依存関係の解決を続行する。

  4. ``file(GET_RUNTIME_DEPENDENCIES)`` コマンドは、プラットフォームごとのリンク規則に従って依存関係（ライブラリ）を探す。

  5. 依存関係（ライブラリ）が見つかり、その絶対パスが ``POST_INCLUDE_REGEXES`` または ``POST_INCLUDE_FILES`` のいずれかのエントリを満足したら、その絶対パスを解決した依存関係のリストに追加し、``file(GET_RUNTIME_DEPENDENCIES)`` コマンドは再帰的に依存関係を解決していく。それに対して依存関係（ライブラリ）が見つからなかったらステップ 6 へ進む。

  6. 依存関係（ライブラリ）が見つかり、その絶対パスが ``POST_EXCLUDE_REGEXES`` または ``POST_EXCLUDE_FILES`` のいずれかのエントリを満足していたら、その絶対パスは解決した依存関係のリストには追加せす、依存関係の解決を停止する。

  7. 依存関係（ライブラリ）が見つかり、その絶対パスが ``POST_INCLUDE_REGEXES`` や ``POST_INCLUDE_FILES`` や ``POST_EXCLUDE_REGEXES`` や ``POST_EXCLUDE_FILES`` のいずれのエントリを満足していなければ、その絶対パスを解決した依存関係のリストに追加し、``file(GET_RUNTIME_DEPENDENCIES)``  コマンドは再帰的に依存関係を解決していく。

  この依存関係を解決するステップには、プラットフォームごとに異なる処理があります。
  ここでは、その詳細について説明します。

  ホストが Linux 系プラットフォームの場合、依存関係（ライブラリ）の解決は次のように処理します：

  1. 依存元のファイルに ``RUNPATH`` のエントリが無く、依存先のライブラリが ``RPATH`` のいずれかのディレクトリか、またはその親ディレクトリの順で存在する場合、その依存関係（ライブラリ）は解決されたものとする。
  2. それ以外で、依存元のファイルに ``RUNPATH`` のエントリが有り、依存先のライブラリがそのエントリのいずれかに存在している場合、その依存関係（ライブラリ）は解決されたものとする。
  3. それ以外で、依存先のライブラリが ``ldconfig`` が返すディレクトリのいずれかに存在している場合、その依存関係（ライブラリ）は解決されたものとする。
  4. それ以外で、依存先のライブラリが ``DIRECTORIES`` のエントリのいずれかに存在している場合、その依存関係（ライブラリ）は解決されたものとする。
     この場合は警告が発行される（``DIRECTORIES`` のエントリのいずれかでライブラリが見つかったということは、依存元のファイルが不完全であることを意味するため）。
  5. それ以外は、依存関係は未解決であるとする。

  ホストが Windows 系プラットフォームの場合、依存関係（ライブラリ）の解決は次のように処理します：

  1. DLL dependency names are converted to lowercase for matching filters.
     Windows DLL names are case-insensitive, and some linkers mangle the case of the DLL dependency names.
     However, this makes it more difficult for ``PRE_INCLUDE_REGEXES``, ``PRE_EXCLUDE_REGEXES``, ``POST_INCLUDE_REGEXES``, and ``POST_EXCLUDE_REGEXES`` to properly filter DLL names - every regex would have to check for both uppercase and lowercase letters.  For example:

     .. code-block:: cmake

       file(GET_RUNTIME_DEPENDENCIES
         # ...
         PRE_INCLUDE_REGEXES "^[Mm][Yy][Ll][Ii][Bb][Rr][Aa][Rr][Yy]\\.[Dd][Ll][Ll]$"
         )

     Converting the DLL name to lowercase allows the regexes to only match lowercase names, thus simplifying the regex.
     For example:

     .. code-block:: cmake

       file(GET_RUNTIME_DEPENDENCIES
         # ...
         PRE_INCLUDE_REGEXES "^mylibrary\\.dll$"
         )

     This regex will match ``mylibrary.dll`` regardless of how it is cased, either on disk or in the depending file. (For example, it will match ``mylibrary.dll``, ``MyLibrary.dll``, and ``MYLIBRARY.DLL``.)

     .. versionchanged:: 3.27

       The conversion to lowercase only applies while matching filters.
       Results reported after filtering case-preserve each DLL name as it is found on disk, if resolved, and otherwise as it is referenced by the dependent binary.

       Prior to CMake 3.27, the results were reported with lowercase DLL file names, but the directory portion retained its casing.

  2. (**Not yet implemented**) If the depending file is a Windows Store app,
     and the dependency is listed as a dependency in the application's package
     manifest, the dependency is resolved to that file.

  3. Otherwise, if the library exists in the same directory as the depending
     file, the dependency is resolved to that file.

  4. Otherwise, if the library exists in either the operating system's
     ``system32`` directory or the ``Windows`` directory, in that order, the
     dependency is resolved to that file.

  5. Otherwise, if the library exists in one of the directories specified by
     ``DIRECTORIES``, in the order they are listed, the dependency is resolved
     to that file. In this case, a warning is not issued, because searching
     other directories is a normal part of Windows library resolution.

  6. Otherwise, the dependency is unresolved.

  ホストが Apple 系プラットフォームの場合、依存関係（ライブラリ）の解決は次のように処理します：

  1. If the dependency starts with ``@executable_path/``, and an
     ``EXECUTABLES`` argument is in the process of being resolved, and
     replacing ``@executable_path/`` with the directory of the executable
     yields an existing file, the dependency is resolved to that file.

  2. Otherwise, if the dependency starts with ``@executable_path/``, and there
     is a ``BUNDLE_EXECUTABLE`` argument, and replacing ``@executable_path/``
     with the directory of the bundle executable yields an existing file, the
     dependency is resolved to that file.

  3. Otherwise, if the dependency starts with ``@loader_path/``, and replacing
     ``@loader_path/`` with the directory of the depending file yields an
     existing file, the dependency is resolved to that file.

  4. Otherwise, if the dependency starts with ``@rpath/``, and replacing
     ``@rpath/`` with one of the ``RPATH`` entries of the depending file
     yields an existing file, the dependency is resolved to that file.
     Note that ``RPATH`` entries that start with ``@executable_path/`` or
     ``@loader_path/`` also have these items replaced with the appropriate
     path.

  5. Otherwise, if the dependency is an absolute file that exists,
     the dependency is resolved to that file.

  6. Otherwise, the dependency is unresolved.

  このコマンドは、依存関係の解決にどのようなツールを使うかを決定する CMake 変数をいくつかサポートしています：

  .. variable:: CMAKE_GET_RUNTIME_DEPENDENCIES_PLATFORM

    ファイルがビルドされたオペレーティング・システムと実行形式を指定します。この変数は次のいずれかの値になります：

    * ``linux+elf``
    * ``windows+pe``
    * ``macos+macho``

    この変数が指定されない場合は、CMake が実行環境から自動的に決定します。

  .. variable:: CMAKE_GET_RUNTIME_DEPENDENCIES_TOOL

    依存関係の解決で使用するツールを指定します。
    CMake 変数の :variable:`CMAKE_GET_RUNTIME_DEPENDENCIES_PLATFORM` の値に応じて、次のいずれかの値になります：

    ================================================== ==============================================
     ``CMAKE_GET_RUNTIME_DEPENDENCIES_PLATFORM`` の値   ``CMAKE_GET_RUNTIME_DEPENDENCIES_TOOL`` の値
    ================================================== ==============================================
    ``linux+elf``                                      ``objdump``
    ``windows+pe``                                     ``objdump`` または ``dumpbin``
    ``macos+macho``                                    ``otool``
    ================================================== ==============================================

    この変数が指定されない場合は、CMake が実行環境から自動的に決定します。

  .. variable:: CMAKE_GET_RUNTIME_DEPENDENCIES_COMMAND

    依存関係の解決で使用するツールのパスを指定します。
    これは ``objdump`` または ``dumpbin`` または ``otool`` の実パスです。

    この変数が指定されない場合は、CMake 変数の ``CMAKE_OBJDUMP`` がセットされていたらその値を使い、それ以外は CMake が実行環境から自動的に決定します。

    .. versionadded:: 3.18
      CMake 変数の ``CMAKE_OBJDUMP`` がセットされていたら、それを使うようになった。

ファイルに書き込む
^^^^^^^^^^^^^^^^^^

.. signature::
  file(WRITE <filename> <content>...)
  file(APPEND <filename> <content>...)

  ``<content>`` を ``<filename>`` というファイルに書き込みます。
  ``<filename>`` が存在していない場合は、書き込む前に作成します。
  ``<filename>`` が既に存在している場合、``WRITE`` サブコマンドはそのファイルの内容を ``<content>`` で上書きし、``APPEND`` サブコマンドはその内容の最後に ``<content>`` を書き込みます。
  ``<filename>`` の中に存在していないディレクトリがあれば、全て作成します。

  ``<filename>`` がビルド時の入力ファイルになる時、その内容が変更されている場合にだけ :command:`configure_file` コマンドを使って更新します。

.. signature::
  file(TOUCH [<files>...])
  file(TOUCH_NOCREATE [<files>...])

  .. versionadded:: 3.12

  ファイルが存在していない場合は、空のファイルを作成します。
  ファイルが既に存在している場合は、このコマンドを呼び出した時の日時でファイルのタイムスタンプ（アクセス日時 および/または 変更日時）を更新します。

  ``TOUCH_NOCREATE`` のサブコマンドは、ファイルが存在している場合は ``touch`` し、ファイルが存在していない場合は何もしません。

  すなわち ``TOUCH`` と ``TOUCH_NOCREATE`` のサブコマンドは、既存のファイルの内容を変更しません。

.. signature::
  file(GENERATE [...])

  :manual:`ジェネレータ <cmake-generators(7)>` が生成したビルドシステムのデータを出力ファイルに書き込みます。
  あるいは、オプションとして受け取ったデータ [#content_of_file]_ から :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を評価して、その結果を出力ファイルに書き込みます。

  .. code-block:: cmake

    file(GENERATE OUTPUT <output-file>
         <INPUT <input-file>|CONTENT <content>>
         [CONDITION <expression>] [TARGET <target>]
         [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS |
          FILE_PERMISSIONS <permissions>...]
         [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])

  指定できるオプションは次のとおりです:

    ``CONDITION <condition>``
      ``<condition>`` が ``TRUE`` の場合にだけ、特定のビルドシステムを含まれる出力ファイルを作成する。
      この ``<condition>`` には、ジェネレータ式を評価したあとに ``0`` または ``1`` のどちらかが格納される。

    ``CONTENT <content>``
      ここで明示的に与えられた ``<content>`` をビルドシステムを生成する時の入力データとして使う。

    ``INPUT <input-file>``
      ``<input-file>`` をビルドシステムを生成する時の入力ファイルとして使う。

      .. versionchanged:: 3.10
        ``<input-file>`` が相対パスを含んでいる場合は CMake 変数の :variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとしてパスを計算するようになった。
        :policy:`CMP0070` のポリシーも参照して下さい。

    ``OUTPUT <output-file>``
      作成する出力ファイル名を指定する。
      :genex:`$<CONFIG>` 等のジェネレータ式を使って、ジェネレータ固有の出力ファイルを指定できる。
      生成されたデータが同一である場合にのみ、複数のビルドシステムで同じ出力ファイルを作成することが可能である。
      それ以外の場合 ``<output-file>`` はビルドシステムごとに重複しないファイル名が付与される。

      .. versionchanged:: 3.10
        ジェネレータ式を評価したあと、``<output-file>`` が相対パスを含んでいる場合は CMake 変数の :variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリとしてパスを計算するようになった。
        :policy:`CMP0070` のポリシーも参照して下さい。

    ``TARGET <target>``
      .. versionadded:: 3.19

      ジェネレータ式を評価する際に必要となるターゲットを指定する（たとえば :genex:`$<COMPILE_FEATURES:...>`、:genex:`$<TARGET_PROPERTY:prop>` など）。

    ``NO_SOURCE_PERMISSIONS``
      .. versionadded:: 3.20

      作成した出力ファイルのアクセス権限として、デフォルトで標準の 644 (``-rw-r--r--``) が適用される。

    ``USE_SOURCE_PERMISSIONS``
      .. versionadded:: 3.20

      作成した出力ファイルに ``<input-file>`` のアクセス権限を適用する。
      アクセス権限を表す 3つのキーワード（``NO_SOURCE_PERMISSIONS``、``USE_SOURCE_PERMISSIONS``、``FILE_PERMISSIONS``）のいずれも指定されていない場合は、``<input-file>`` のアクセス権限を適用することがデフォルトの動作である。
      主に、この ``USE_SOURCE_PERMISSIONS`` オプションは、``file()`` コマンドを呼び出した側の対応が意図したものであることを明確にする方法として使われる。
      ``INPUT`` オプションなしで、このオプションを指定するとエラーを発行する。

    ``FILE_PERMISSIONS <permissions>...``
      .. versionadded:: 3.20

      ここで指定した ``<permissions>`` を作成した出力ファイルに適用する。

    ``NEWLINE_STYLE <style>``
      .. versionadded:: 3.20

      作成するファイルの改行スタイルを指定する。
      指定可能なスタイルは、改行文字が ``\n`` の場合は ``UNIX`` または ``LF``、 改行文字が ``\r\n`` の場合は ``DOS``、``WIN32`` または ``CRLF`` である。

  ``CONTENT`` と ``INPUT`` オプションはどちらか一つ指定して下さい。
  ``file(GENERATE)`` コマンドを一回呼び出すと、``OUTPUT`` オプションで指定した ``<output-file>`` が作成されます。
  出力ファイルの内容が変更された場合にのみ、ファイルのタイムスタンプが更新されます。

  この ``file(GENERATE)`` コマンドには注意点があります。
  このコマンドはビルドシステムの生成が完了するまで出力ファイルを作成しません。
  さらに ``file(GENERATE)`` コマンドの呼び出しから戻ってきた時点でも、まだ生成したデータは書き込まれていません。
  すなわち、現在のプロジェクトに関連する全ての ``CMakeLists.txt`` ファイルを処理した後に、はじめて書き込まれます。

.. signature::
  file(CONFIGURE OUTPUT <output-file>
       CONTENT <content>
       [ESCAPE_QUOTES] [@ONLY]
       [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])
  :target: CONFIGURE

  .. versionadded:: 3.18

  ``CONTENT`` オプションで指定した入力データ ``<content>`` から出力ファイルを作成します。その際は、``<content>`` に含まれている ``@VAR@`` や ``${VAR}`` で参照される変数の値を置き換えます。
  この置き換えは :command:`configure_file` コマンドが採用しているルールに従います。
  :command:`configure_file` コマンドに準拠させているため、``OUTPUT`` と ``CONTENT`` オプションではジェネレータ式をサポートしていないので注意して下さい。

  指定できるオプションは次のとおりです:

    ``OUTPUT <output-file>``
      作成する出力ファイル名を指定する。
      ``<output-file>`` が相対パスを含んでいる場合は CMake 変数の :variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリとして使う。
      ジェネレータ式を使った ``<output-file>`` の指定はサポートしない。

    ``CONTENT <content>``
      ここで明示的に与えられた ``<content>`` をビルドシステム生成の入力データとして使う。
      ジェネレータ式を使った ``<content>`` の処理はサポートしない。

    ``ESCAPE_QUOTES``
      置き換えたあとにクォート文字をバックスラッシュでエスケープする（C言語方式）。

    ``@ONLY``
      変数の値の置き換えを ``@VAR@`` だけに制限する。
      これは ``${VAR}`` を使うスクリプトを構成する際に便利である。

    ``NEWLINE_STYLE <style>``
      作成するファイルの改行スタイルを指定する。
      指定可能なスタイルは、改行文字が ``\n`` の場合は ``UNIX`` または ``LF``、 改行文字が ``\r\n`` の場合は ``DOS``、``WIN32`` または ``CRLF`` である。

ファイルシステムを使う
^^^^^^^^^^^^^^^^^^^^^^

.. signature::
  file(GLOB <variable>
       [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
       [<globbing-expressions>...])
  file(GLOB_RECURSE <variable> [FOLLOW_SYMLINKS]
       [LIST_DIRECTORIES true|false] [RELATIVE <path>] [CONFIGURE_DEPENDS]
       [<globbing-expressions>...])

  指定した ``<globbing-expressions>`` （グロブ式）にマッチするファイルのリストを生成し、そのリストを ``<variable>`` に格納します。
  この ``<globbing-expressions>`` は正規表現に似ていますが、より単純です。
  ``RELATIVE`` オプションを指定すると、生成したファイルのパスは ``<path>`` をベース・ディレクトリとした相対パスに変換されます。

  .. versionchanged:: 3.6
    生成したファイルのパスはアルファベット順にリストに格納されるようになった。

  ホストが Windows や macOS 系のプラットフォームの場合、それぞれのファイルシステムがファイル名の大文字と小文字を区別できるとしても、このコマンドは無視します（つまり、コマンドを実行する前にファイル名と ``<globbing-expression>`` の両方を全て小文字に変換します）。
  それ以外のターゲットでは大文字と小文字を区別します。

  .. versionadded:: 3.3
    この ``GLOB`` サブコマンドは、デフォルトでディレクトリもリスト化するようになった。
    ただし ``LIST_DIRECTORIES`` オプションを ``FALSE`` にした場合は除く。

  .. versionadded:: 3.12
    ``CONFIGURE_DEPENDS`` オプションを指定すると、ビルド時にフラグが付いた ``GLOB`` サブコマンドを再実行できるようになった。
    再実行した結果、リストの内容が更新されたら、ビルドシステムを再生成する。

  .. note::
    この ``GLOB`` サブコマンドを使ってソースツリーから入力ファイルのリストを得ることは推奨しません。
    このサブコマンドを使ってソース・ファイルを追加したり削除したとしても、``CMakeLists.txt`` 自身は変更されないため、一度生成されたビルドシステムは、いつビルドシステムの再生成を CMake に要求すべきか判断できないからです。
    また ``CONFIGURE_DEPENDS`` オプションは全てのジェネレータ上で動作するとは限らず、将来、そのオプションをサポートしない新しいジェネレータが追加された場合、このコマンドを使うプロジェクトは互換性を維持できなくなります。
    仮に ``CONFIGURE_DEPENDS`` オプションが確実に動作したとしても、依然として、ビルドシステムを再生成するたびにファイルシステムを走査するというコストがつきまといます。

  ``<globbing-expressions>`` の例：

  ============== =================================================================
  ``*.cxx``      拡張子が ``cxx`` である全てのファイルにマッチする
  ``*.vt?``      拡張子が ``vta`` , ..., ``vtz`` である全てのファイルにマッチする
  ``f[3-5].txt`` ``f3.txt`` または ``f4.txt`` または ``f5.txt`` にマッチする
  ============== =================================================================

  ``GLOB_RECURSE`` サブコマンドは、``<globbing-expression>`` にマッチするディレクトリ下の全てのサブディレクトリを走査してマッチするものをチェックします。
  サブディレクトリがシンボリックリンクの場合、``FOLLOW_SYMLINKS`` オプションを指定するか、:policy:`CMP0009` ポリシーが ``OLD``  の場合にだけチェックします。

  .. versionadded:: 3.3
    デフォルトで、``GLOB_RECURSE`` サブコマンドはリストにはディレクトリを含めないようになった。
    ただし ``LIST_DIRECTORIES`` オプションを ``TRUE`` にした場合は除く。
    ``FOLLOW_SYMLINKS`` オプションを指定するか、または :policy:`CMP0009` ポリシーを ``OLD`` にすると、``GLOB_RECURSE`` サブコマンド [#maybe_misprint_LIST_DIRECTORIES]_ はシンボリックをディレクトリとして扱う。

  再帰的な ``<globbing-expressions>`` の例：

  ============== =========================================================================
  ``/dir/*.py``  ``/dir`` とそのサブディレクトリ下にある全ての python ファイルにマッチする
  ============== =========================================================================

.. signature::
  file(MAKE_DIRECTORY [<directories>...])

  指定した ``<directories>`` と、必要に応じて、その親ディレクトリを作成します。

.. signature::
  file(REMOVE [<files>...])
  file(REMOVE_RECURSE [<files>...])

  指定した ``<files>`` をファイルシステムから削除します。
  ``REMOVE_RECURSE`` サブコマンドは、指定した ``<files>`` とそのサブディレクトリ（空ではないディレクトリを含む）を全て削除します。
  指定したファイルが存在していなくても、エラーを発行しません。
  ``<files>`` に相対パスが含まれている場合は、現在のソース・ディレクトリをベース・ディレクトリとしてパスを評価します。

  .. versionchanged:: 3.15
    ``<files>`` の中に空文字のパスが含まれている場合は無視するが、警告を発行するようになった。
    以前のバージョンでは、空の文字列を現在のディレクトリとし、かつ相対パスのベース・ディレクトリであるとしてパスを評価し、該当するファイルを削除していた。

.. signature::
  file(RENAME <oldname> <newname> [RESULT <result>] [NO_REPLACE])

  ファイルシステム上のファイルまたはディレクトリを ``<oldname>`` から ``<newname>`` へ移動します（移送先をアトミックに置き換えます）。

  指定できるオプションは次のとおりです:

    ``RESULT <result>``
      .. versionadded:: 3.21

      この操作が成功したら ``<result>`` という変数に ``0`` をセットし、それ以外はエラーメッセージをセットする。
      この ``RESULT`` オプションを指定しない場合に操作が失敗したら、エラーを発行する。

    ``NO_REPLACE``
      .. versionadded:: 3.21

      ``<newname>`` が既に存在している場合は置き換えない。
      その際に ``RESULT <result>`` オプションを指定していたら、``<result>`` 変数には ``NO_REPLACE`` をセットする。
      それ以外は、エラーを発行する。

.. signature::
  file(COPY_FILE <oldname> <newname>
       [RESULT <result>]
       [ONLY_IF_DIFFERENT]
       [INPUT_MAY_BE_RECENT])

  .. versionadded:: 3.21

  ファイルシステム上のファイルを ``<oldname>`` から ``<newname>`` にコピーします。
  ディレクトリのコピーはサポートしていません。
  シンボリックリンクは無視し、そのリンクが指す ``<oldfile>`` の内容を ``<newname>`` という新しいファイルを作成して書き込みます。

  指定できるオプションは次のとおりです:

    ``RESULT <result>``
      この操作が成功したら ``<result>`` という変数に ``0`` をセットし、それ以外はエラーメッセージをセットする。
      この ``RESULT`` オプションを指定しない場合に操作が失敗したら、エラーを発行する。

    ``ONLY_IF_DIFFERENT``
      ``<newname>`` が既に存在し、その内容が ``<oldname>`` の内容と同じ場合は置き換えない（これにより ``<newname>`` のタイムスタンプが更新されずに済む）。

    ``INPUT_MAY_BE_RECENT``
      .. versionadded:: 3.26

      入力ファイルが最近作成された旨を CMake に知らせる。
      このオプションは、ホストが Windows 系プラットフォームの場合にだけ意味を持つ（ファイルの作成直後は、そのファイルにアクセスできない場合があるため）。
      このオプションを指定すると、ファイルへのアクセスが拒否された場合に CMake はファイルの読み取りを数回繰り返す。

  このサブコマンドには、``COPYONLY`` オプションを指定した :command:`configure_file` コマンドと類似点がいくつかあります。
  重要な違いは、:command:`configure_file` コマンドが入力ファイル上で依存関係を生成するので、もし入力ファイルが変更されていたら、もう一度コマンドを再実行するという点です。
  これに対して、この ``file(COPY_FILE)`` サブコマンドは依存関係を生成しません。

  ファイルのコピー機能を拡張する、この下の :command:`file(COPY)` サブコマンドも参照して下さい。

.. signature::
  file(COPY [...])
  file(INSTALL [...])

  ``COPY`` サブコマンドはいくつかのファイル、ディレクトリ、そしてシンボリックリンクをコピー先にコピーします。
  コピー元が相対パスの場合は :variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとして、コピー先が相対パスの場合は :variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリとして、それぞれパスを評価します。
  コピーを実行すると、コピー元のタイムスタンプを保持し、コピー先が同じタイムスタンプであったら最適化します。
  アクセス権を指定するか、または ``NO_SOURCE_PERMISSIONS`` オプションを指定する場合を除き、コピー元とコピー先のアクセス権限は同じになります（デフォルトは動きは ``USE_SOURCE_PERMISSIONS`` なので）。

  .. code-block:: cmake

    file(<COPY|INSTALL> <files>... DESTINATION <dir>
         [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS]
         [FILE_PERMISSIONS <permissions>...]
         [DIRECTORY_PERMISSIONS <permissions>...]
         [FOLLOW_SYMLINK_CHAIN]
         [FILES_MATCHING]
         [[PATTERN <pattern> | REGEX <regex>]
          [EXCLUDE] [PERMISSIONS <permissions>...]] [...])

  .. note::

    単純なファイルをコピーするのであれば、:command:`file(COPY_FILE)` サブコマンドの方を使ったほうが簡単かもしれません。

  .. versionadded:: 3.15
    ``FOLLOW_SYMLINK_CHAIN`` オプションを指定すると、``COPY`` サブコマンドは実ファイルが見つかるまで指定したパスでシンボリックリンクの解決を再帰的に試み、解決できたら、その実ファイルに対するシンボリックリンクを作成する。
    作成するシンボリックリンクからディレクトリ部分を取り除きファイル名だけにすることで、新しく作成したシンボリックリンクは同じディレクトリにあるファイルを指すことになる。
    この機能は、ライブラリがバージョン番号を持つシンボリックリンクのチェインとしてインストールされている（たとえば、バージョンが付いていないシンボリックリンクが、バージョンが付いたより具体的なシンボリックリンクを指す等）一部の Unix 系のシステムの場合に便利である。
    ``FOLLOW_SYMLINK_CHAIN`` オプションは、このようなチェイン化した全てのシンボリックリンクと、ライブラリ自身をコピー先のディレクトリへコピーする。
    例えば、次のようなファイルがあるとする：

      * ``/opt/foo/lib/libfoo.so.1.2.3``
      * ``/opt/foo/lib/libfoo.so.1.2 -> libfoo.so.1.2.3``
      * ``/opt/foo/lib/libfoo.so.1 -> libfoo.so.1.2``
      * ``/opt/foo/lib/libfoo.so -> libfoo.so.1``

    そして次のコマンドを実行する：

    .. code-block:: cmake

      file(COPY /opt/foo/lib/libfoo.so DESTINATION lib FOLLOW_SYMLINK_CHAIN)

    これにより、``libfoo.so.1.2.3`` に対する全てのシンボリックリンクと ``libfoo.so.1.2.3`` 自身を ``lib`` ディレクトリにコピーする。

  アクセス権限、``FILES_MATCHING``、``PATTERN``、``REGEX``、そして ``EXCLUDE`` オプションについては :command:`install(DIRECTORY)` コマンドを参照して下さい。
  オプションを使用してファイルのサブセットを選択した場合でも、ディレクトリをコピーするとその中のディレクトリ構造が保持されます。

  ``INSTALL`` サブコマンドは ``COPY`` サブコマンドとは少し異なり、コマンド実行中にステータス・メッセージを表示し、コピー先のアクセス権限は ``NO_SOURCE_PERMISSIONS`` がデフォルトです。
  なお :command:`install` コマンドによって作成されたインストール・スクリプトは、この ``INSTALL`` サブコマンド（と内部使用のためドキュメント化されていないオプション）を使用します。

  .. versionchanged:: 3.22

    :envvar:`CMAKE_INSTALL_MODE` という環境変数は :command:`file(INSTALL)` コマンドのデフォルトのコピー操作を上書きする。

.. signature::
  file(SIZE <filename> <variable>)

  .. versionadded:: 3.14

  ``<filename>`` のファイルサイズを調べ、``<variable>`` という変数に格納します。
  この時 ``<filename>`` はファイルを指すパスとして有効であり、実際に読み取りできることが条件です。

.. signature::
  file(READ_SYMLINK <linkname> <variable>)

  .. versionadded:: 3.14

  シンボリックリンクの ``<linkname>`` をシステムに問い合わせて、それが指すパスを ``<variable>`` という変数に格納します。
  ``<linkname>`` が存在しない、またはシンボリックリンクではない場合は Fatal エラーを発行します。

  このコマンドは「生の」シンボリックリンクのパスを返し、相対パスとしては未評価であるという点に注意して下さい。
  次は、このコマンドの結果から絶対パスを取得する例です：

  .. code-block:: cmake

    set(linkname "/path/to/foo.sym")
    file(READ_SYMLINK "${linkname}" result)
    if(NOT IS_ABSOLUTE "${result}")
      get_filename_component(dir "${linkname}" DIRECTORY)
      set(result "${dir}/${result}")
    endif()

.. signature::
  file(CREATE_LINK <original> <linkname>
       [RESULT <result>] [COPY_ON_ERROR] [SYMBOLIC])

  .. versionadded:: 3.14

  ``<original>`` を指す ``<linkname>`` というリンクを作成します。
  デフォルトではハードリンクになりますが、``SYMBOLIC`` オプションを指定するとシンボリックリンクになります。
  ハードリンクにするには ``original`` がファイルシステム上に存在し、ディレクトリではなくファイルであることが条件です。
  もし ``<linkname>`` が既に存在していたら、それを上書きします。

  ``<result>`` という変数を指定すると、このコマンドの実行ステータスを格納します。
  コマンドが成功したら ``0``、それ以外はエラーメッセージが格納されます。
  ``RESULT`` オプションを指定しない場合に操作が失敗したら、Fatal エラーを発行します。

  ``COPY_ON_ERROR`` オプションを指定すると、リンクの作成に失敗した場合、そのフォールバック処理としてファイル自身をコピーします。
  これは、たとえば ``<original>`` と ``<linkname>`` がそれぞれ異なるドライブまたはマウント・ポイント上にあるため、ハードリンクを作成できないようなケースで便利なオプションです。

.. signature::
  file(CHMOD <files>... <directories>...
       [PERMISSIONS <permissions>...]
       [FILE_PERMISSIONS <permissions>...]
       [DIRECTORY_PERMISSIONS <permissions>...])

  .. versionadded:: 3.19

  指定した ``<files>...`` および ``<directories>...`` のアクセス権限をセットします。
  セットできる権限（キーワード）は ``OWNER_READ``、``OWNER_WRITE``、``OWNER_EXECUTE``、``GROUP_READ``、``GROUP_WRITE``、``GROUP_EXECUTE``、 ``WORLD_READ``、``WORLD_WRITE``、``WORLD_EXECUTE``、``SETUID``、そして ``SETGID`` です。

  各キーワードの有効な組み合わせは次のとおりです：

    ``PERMISSIONS``
      全てのキーワードを変更する。

    ``FILE_PERMISSIONS``
      ファイルだけ変更する。

    ``DIRECTORY_PERMISSIONS``
      ディレクトリだけ変更する。

    ``PERMISSIONS`` と ``FILE_PERMISSIONS``
      ``FILE_PERMISSIONS`` でファイルの ``PERMISSIONS`` を上書きする。

    ``PERMISSIONS`` と ``DIRECTORY_PERMISSIONS``
      ``DIRECTORY_PERMISSIONS`` でディレクトリの ``PERMISSIONS`` を上書きする。

    ``FILE_PERMISSIONS`` と ``DIRECTORY_PERMISSIONS``
      ファイルの場合は ``FILE_PERMISSIONS`` を使い、ディレクトリの場合は ``DIRECTORY_PERMISSIONS`` を使う

.. signature::
  file(CHMOD_RECURSE <files>... <directories>...
       [PERMISSIONS <permissions>...]
       [FILE_PERMISSIONS <permissions>...]
       [DIRECTORY_PERMISSIONS <permissions>...])

  .. versionadded:: 3.19

  基本は :cref:`CHMOD` サブコマンドと同じですが、``<directories>...`` 配下に存在するファイルとディレクトリのアクセス権限を再帰的に（一括で）セットします。


パスを評価する
^^^^^^^^^^^^^^

.. signature::
  file(REAL_PATH <path> <out-var> [BASE_DIRECTORY <dir>] [EXPAND_TILDE])

  .. versionadded:: 3.19

  シンボリックリンクが解決されたファイルやディレクトリの絶対パスを計算します。
  指定できるオプションは次のとおりです：

    ``BASE_DIRECTORY <dir>``
      指定した ``<path>`` が相対パスの場合、``<dir>`` をベース・ディレクトリとしてパスを評価する。
      ``<dir>`` を指定しない場合は、デフォルトのベース・ディレクトリとして :variable:`CMAKE_CURRENT_SOURCE_DIR` を使用する。

    ``EXPAND_TILDE``
      .. versionadded:: 3.21

      ``<path>`` が ``~`` （チルダ）か、または ``~/`` で始まる場合、``~`` をユーザのホーム・ディレクトリに置き換える。
      ホーム・ディレクトリへのパスは実行時の環境変数から取得する。
      ホストが Windows 系のプラットフォームの場合、``USERPROFILE`` という環境変数から取得し、この変数が定義されていない場合は ``HOME`` 環境変数から取得する。
      それ以外のプラットフォームは全て ``HOME`` 環境変数から取得する。

  .. versionchanged:: 3.28

    全てのシンボリックリンクが、``../`` の部分を評価する前に解決されるようになった。
    :policy:`CMP0152` のポリシーも参照して下さい。

.. signature::
  file(RELATIVE_PATH <variable> <directory> <file>)

  ``<directory>`` から ``<file>`` への相対パスを計算して、それを ``<variable>`` という変数に格納する。

.. signature::
  file(TO_CMAKE_PATH "<path>" <variable>)
  file(TO_NATIVE_PATH "<path>" <variable>)

  ``TO_CMAKE_PATH`` サブコマンドは、ホストにネィティブな ``<path>`` をスラッシュ（"``/``"）を使った CMake スタイルのパスに変換します。
  ``<path>`` には単一のパス、または ``$ENV{PATH}`` のような環境変数の検索パスを渡すことができます。
  この検索パスは、セミコロン文字（"``;``"）で区切られた CMake スタイルのパスを要素とするリストに変換されます。

  ``TO_NATIVE_PATH`` サブコマンドは、CMake スタイルの ``<path>`` をホストのプラットフォーム固有のスラッシュ（Windows 系のプラットフォームの場合は ``\``、それ以外のプラットフォームは ``/``）を含んだネィティブなパスに変換します。

  ``<path>`` が一個の引数として扱われるように、必ず ``<path>`` を二重引用符で囲んで下さい（例: ``"<path>"``）。

ファイルを転送する
^^^^^^^^^^^^^^^^^^

.. signature::
  file(DOWNLOAD <url> [<file>] [<options>...])
  file(UPLOAD <file> <url> [<options>...])

  ``DOWNLOAD`` サブコマンドは、指定した ``<url>`` をローカルの ``<file>`` にダウンロードします。
  ``UPLOAD`` サブコマンドは、ローカルの ``<file>`` を指定した ``<url>`` へアップロードします。

  .. versionadded:: 3.19
    ``file(DOWNLOAD)`` で ``<file>`` が指定されなかったら、ダウロードしてもローカルには保存しないようになった。
    これは、ファイルを保存すること無く、単にダウンロードできるかどうか知りたい場合（たとえば、``<url>`` が存在しているかどうかを確認する場合）に便利である。

  両サブコマンドに共通のオプションは次のとおりです：

    ``INACTIVITY_TIMEOUT <seconds>``
      アクティビティがない状態が一定時間（``<seconds>``）続いたあとに操作を終了する。

    ``LOG <variable>``
      操作中のログを ``<variable>`` に格納する。

    ``SHOW_PROGRESS``
      操作が完了するまで、進行状況をステータス・メッセージに出力する。

    ``STATUS <variable>``
      操作の終了ステータスを ``<variable>`` に格納する。
      このステータスは ``;`` で区切ったリスト（サイズは 2）で、最初の要素は終了ステータス（数値）で、二番目の要素はエラーメッセージ（文字列）である。
      終了ステータスが ``0`` の場合はエラーは無いことを示す。

    ``TIMEOUT <seconds>``
      操作を開始してから指定した ``<seconds>`` 時間が経過したら操作を終了する。

    ``USERPWD <username>:<password>``
      .. versionadded:: 3.7

      操作で使用するユーザ名とパスワードを設定する。

    ``HTTPHEADER <HTTP-header>``
      .. versionadded:: 3.7

      各操作で使用する HTTP ヘッダを設定する。この ``HTTPHEADER`` オプションは繰り返し指定できる。たとえば：

      .. code-block:: cmake

        file(DOWNLOAD <url>
             HTTPHEADER "Authorization: Bearer <auth-token>"
             HTTPHEADER "UserAgent: Mozilla/5.0")

    ``NETRC <level>``
      .. versionadded:: 3.11

      ``.netrc`` ファイルを使用するかどうかを指定する。
      このオプションを指定しない場合は、代わりに CMake 変数の :variable:`CMAKE_NETRC` の値を使用する。

      有効な ``<level>`` の値は次の通り：

        ``IGNORED``
          ``.netrc`` ファイルを無視する。
          これがデフォルトの値。

        ``OPTIONAL``
          ``.netrc`` ファイルがオプションで、``<url>`` 内の情報を優先する。 
          ``<url>`` の中で指定されていない情報があれば、 ``.netfc`` ファイルを読み込む。

        ``REQUIRED``
          ``.netrc`` ファイルの情報を優先し、``<url>`` の中の情報を無視する。

    ``NETRC_FILE <file>``
      .. versionadded:: 3.11

      ``NETRC`` オプションに ``OPTIONAL`` またはr ``REQUIRED`` を指定した場合、ホーム・ディレクトリにある ``.netrc`` ファイルの代わり使う ``<file>`` を指定する。
      このオプションを指定しない場合、 代わりに CMake 変数の :variable:`CMAKE_NETRC_FILE` に指定したファイルを使用する。

    ``TLS_VERIFY <ON|OFF>``
      ``https://`` 系の URL で使うサーバ証明書を検証するかどうかを指定する。
      「*検証しない*」がデフォルト。
      このオプションを指定しない場合、 代わりに CMake 変数の :variable:`CMAKE_TLS_VERIFY` に指定した値を使用する。

      .. versionadded:: 3.18
        ``file(UPLOAD)`` に対するサポートを追加した。

    ``TLS_CAINFO <file>``
      ``https://`` 系の URL で使う独自の認証局ファイルを指定する。
      このオプションを指定しない場合、 代わりに CMake 変数の :variable:`CMAKE_TLS_CAINFO` に指定した値を使用する。

      .. versionadded:: 3.18
        ``file(UPLOAD)`` に対するサポートを追加した。

  ``https://`` 系の URL を使う場合、CMake は OpenSSL サポート付きでビルドする必要があります。
  ``TLS/SSL`` の証明書は、デフォルトではチェックしません。
  ``TLS_VERIFY`` オプションを ``ON`` にすると証明書をチェックします。

  ``DOWNLOAD`` サブコマンド向けの追加のオプションは次のとおりです：

    ``EXPECTED_HASH <algorithm>=<value>``
      ダウンロードしたファイルのハッシュ値が期待値 ``<value>`` と合致するか検証する（``<algorithm>`` には :cref:`<HASH>` でサポートしているアルゴリズムの一つを指定する）。
      ダウンロードする前に既にファイルが存在し、そのハッシュ値が合致する場合は、ダウンロードをスキップする。
      ダウンロードする前に既にファイルが存在し、そのハッシュ値が合致しない場合は、もう一度ダウンロードする。
      ダウンロードしたファイルのハッシュ値が合致しない場合、``DOWNLOAD`` コマンドの操作はエラーになる。
      ``DOWNLOAD`` サブコマンドで、このオプションを指定しながら ``<file>`` を指定していない場合はエラーを発行する。

    ``EXPECTED_MD5 <value>``
      ``EXPECTED_HASH MD5=<value>`` オプションの短縮形。
      ``DOWNLOAD`` サブコマンドで、このオプションを指定しながら ``<file>`` を指定していない場合はエラーを発行する。

    ``RANGE_START <value>``
      .. versionadded:: 3.24

      ダウンロードする範囲の開始オフセットをバイト単位で指定する。
      ファイルの先頭から ``RANGE_END`` までダウンロードする場合は省略できる。

    ``RANGE_END <value>``
      .. versionadded:: 3.24

      ダウンロードする範囲の終端オフセットをバイト単位で指定する。
      ``RANGE_START`` からファイルの終端までダウンロードする場合は省略できる。

ファイルをロックする
^^^^^^^^^^^^^^^^^^^^

.. signature::
  file(LOCK <path> [DIRECTORY] [RELEASE]
       [GUARD <FUNCTION|FILE|PROCESS>]
       [RESULT_VARIABLE <variable>]
       [TIMEOUT <seconds>])

  .. versionadded:: 3.2

  ``DIRECTORY`` オプションを指定しない場合は ``<path>`` のファイルをロックし、それ以外は ``<path>/cmake.lock`` をロックします。
  ファイルは、``GUARD`` オプションで定義したスコープ（デフォルト値は ``PROCESS``）に対してロックされます
  ``RELEASE`` オプションを使うと、明示的にファイルのロックを解除できます。
  ``TIMEOUT`` オプションを指定しない場合、CMake はロックが成功するか、または Fatal エラーが発行されるまで待機します。
  ``TIMEOUT`` オプションに ``0`` をセットすると、ロックは1回だけ試行され、結果は直ぐに報告されます。
  それに対して、``TIMEOUT`` オプションに ``0`` 以外をセットすると、CMake は ``TIMEOUT <seconds>`` の周期でファイルのロックを試みます。
  ``RESULT_VARIABLE`` オプションを指定しない場合は、すべて Fatal エラーとして解釈されます。
  ``RESULT_VARIABLE`` オプションを指定した場合は、成功時は ``0``、失敗したらエラー・メッセージがそれぞれ ``<variable>`` に格納されます。

  あくまでも、このロックは「勧告」レベルであることに注意して下さい。
  他の CMake プロセスが、このロックを尊重することは保証されません。
  このロックは、変更が可能なリソースを共有する2つ以上の CMake のインスタンスと同期するだけです。
  同様のロジックが ``DIRECTORY`` オプションを指定した場合も適用されます。
  親ディレクトリをロックしても、他の ``LOCK`` サブコマンドによるサブディレクトリやファイルのロックを排他することはできません。

  同じファイルを2回ロックすることは許されません。
  指定した ``<path>`` に含まれるディレクトリやファイルが存在していなければ作成します。
  ``RELEASE`` オプションを指定すると、``GUARD`` と ``TIMEOUT`` オプションは無視されます。

アーカイブを扱う
^^^^^^^^^^^^^^^^

.. signature::
  file(ARCHIVE_CREATE OUTPUT <archive>
    PATHS <paths>...
    [FORMAT <format>]
    [COMPRESSION <compression>
     [COMPRESSION_LEVEL <compression-level>]]
    [MTIME <mtime>]
    [VERBOSE])
  :target: ARCHIVE_CREATE
  :break: verbatim

  .. versionadded:: 3.18

  ``<paths>`` にリストされているファイルとディレクトリを使って ``<archive>`` を作成します。
  ``<paths>`` には存在するファイルまたはディレクトリを指定して下さい。ワイルドカードはサポートしていません。

  ``FORMAT <format>`` オプションでアーカイブの種類を指定します。
  ``<format>`` に指定できるアーカイブの種類は ``7zip``、``gnutar``、``pax``、``paxr``、``raw``、そして ``zip`` です。
  ``FORMAT`` オプションを指定しない場合は、デフォルトは ``paxr`` です。

  一部のアーカイブは圧縮の種類を指定できます。
  ``7zip`` と ``zip`` のアーカイブは圧縮も含まれています。
  それ以外はデフォルトで圧縮は行いませんが、``COMPRESSION <compression>`` オプションを使用して圧縮するよう指示できます。
  ``<compression>`` に指定できる圧縮の種類は ``None``、``BZip2``、``GZip``、``XZ``、そして ``Zstd`` です。

  .. versionadded:: 3.19
    圧縮レベルを ``COMPRESSION_LEVEL``  オプションで指定できるようになった。
    ``<compression-level>`` に指定できる値は0〜9の範囲（デフォルトは 0）である。
    ``COMPRESSION_LEVEL`` オプションを指定する場合は ``COMPRESSION`` オプションが必須である。

  .. versionadded:: 3.26
    ``Zstd`` アルゴリズムの ``<compression-level>`` が0〜19の範囲で指定できるようになった。

  .. note::
    ``FORMAT`` に ``raw`` を指定した場合は、``COMPRESSION`` オプションで指定した圧縮タイプで1個だけファイルを圧縮します。

  ``VERBOSE`` オプションを指定すると、アーカイブ操作の冗長なログ出力が有効になります。

  ``MTIME`` オプションを使って、アーカイブしたファイルのタイムスタンプ（変更日時）を指定できます。

.. signature::
  file(ARCHIVE_EXTRACT
    INPUT <archive>
    [DESTINATION <dir>]
    [PATTERNS <patterns>...]
    [LIST_ONLY]
    [VERBOSE]
    [TOUCH])
  :target: ARCHIVE_EXTRACT

  .. versionadded:: 3.18

  ``<archive>`` の内容を展開するか、または一覧表示します。

  アーカイブを展開する先のディレクトリを ``DESTINATION`` オプションで指定できます。
  指定した ``<dir>`` が存在しない場合は作成します。
  ``DESTINATION`` オプションを指定しない場合は :variable:`CMAKE_CURRENT_BINARY_DIR` に展開します。

  指定した ``PATTERNS <patterns>`` を使って、アーカイブからどのファイルやディレクトリを展開または一覧表示するかを選択できます。
  この ``PATTERNS`` オプションはワイルドカードをサポートしています。
  ``PATTERNS`` オプションを指定しない場合は、アーカイブ全体を展開または一覧表示します。

  ``LIST_ONLY`` オプションはアーカイブを展開することなく一覧表示だけします。

  .. versionadded:: 3.24
    ``TOUCH`` オプションは、アーカイブからタイムスタンプを変更することなくそのまま展開する代わりに、アーカイブから展開した時の日時をタイムスタンプにして展開する。

  ``VERBOSE`` オプションを指定すると、アーカイブ操作の冗長なログ出力が有効になります。

.. rubric:: 日本語訳注記

.. [#hint_for_framework_and_bundle_of_ios] 「`Frameworkとは＠Qiita <https://qiita.com/gdate/items/b49ef26824504bb61856#framework%E3%81%A8%E3%81%AF>`_」参照。
.. [#content_of_file] ファイルの「内容」に相当するもの。「データ」はファイルに書き込まれてファイルの「内容」になり、ファイルから読み込んだ内容が「データ」になる。
.. [#maybe_misprint_LIST_DIRECTORIES] おそらく原文の ``LIST_DIRECTORIES treats symlinks ...`` は ``GLOB_RECURSE treats symlinks ...`` の誤植。
