cmake_host_system_information
-----------------------------

ホスト・システムのいろいろな情報を照会する。

概要
^^^^

.. parsed-literal::

  `Query host system specific information`_
    cmake_host_system_information(RESULT <variable> QUERY <key> ...)

  `Query Windows registry`_
    cmake_host_system_information(RESULT <variable> QUERY WINDOWS_REGISTRY <key> ...)

.. _Query host system specific information:

ホスト・システム固有の情報を照会する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  cmake_host_system_information(RESULT <variable> QUERY <key> ...)


:program:`cmake` を実行するホスト・システムのシステム情報を照会します。
問い合わせする情報を選択するために、１個以上の ``<key>`` を指定できます。
照会した値を要素とするリストは ``<variable>`` に格納されます。

``<key>`` には以下の値のいずれかを指定できます：

``NUMBER_OF_LOGICAL_CORES``
  論理コアの数

``NUMBER_OF_PHYSICAL_CORES``
  物理コアの数

``HOSTNAME``
  ホスト名

``FQDN``
  完全就職のドメイン名（FQDN）

``TOTAL_VIRTUAL_MEMORY``
  仮想メモリの合計サイズ（MiB） [#mebibytes]_

``AVAILABLE_VIRTUAL_MEMORY``
  利用可能な仮想メモリのサイズ（MiB） [#mebibytes]_

``TOTAL_PHYSICAL_MEMORY``
  物理メモリの合計サイズ（MiB） [#mebibytes]_

``AVAILABLE_PHYSICAL_MEMORY``
  利用可能な物理メモリのサイズ（MiB） [#mebibytes]_

``IS_64BIT``
  .. versionadded:: 3.10

  プロセッサが 64Bit ならば 1

``HAS_FPU``
  .. versionadded:: 3.10

  プロセッサに浮動小数点演算ユニットがある場合は 1

``HAS_MMX``
  .. versionadded:: 3.10

  プロセッサが MMX 命令をサポートしている場合は 1

``HAS_MMX_PLUS``
  .. versionadded:: 3.10

  プロセッサが拡張 MMX 命令をサポートしている場合は 1

``HAS_SSE``
  .. versionadded:: 3.10

  プロセッサが SSE 命令をサポートしている場合は 1

``HAS_SSE2``
  .. versionadded:: 3.10

  プロセッサが SSE2 命令をサポートしている場合は 1

``HAS_SSE_FP``
  .. versionadded:: 3.10

  プロセッサが SSE EP 命令をサポートしている場合は 1


``HAS_SSE_MMX``
  .. versionadded:: 3.10

  プロセッサが SSE MMX 命令をサポートしている場合は 1

``HAS_AMD_3DNOW``
  .. versionadded:: 3.10

  プロセッサが 3DNow 命令をサポートしている場合は 1

``HAS_AMD_3DNOW_PLUS``
  .. versionadded:: 3.10

  プロセッサが 3DNow+ 命令をサポートしている場合は 1

``HAS_IA64``
  .. versionadded:: 3.10

  x86 のエミュレーションをサポートしている IA64 プロセッサの場合は 1

``HAS_SERIAL_NUMBER``
  .. versionadded:: 3.10

  プロセッサがシリアル番号を持っている場合は 1

``PROCESSOR_SERIAL_NUMBER``
  .. versionadded:: 3.10

  プロセッサのシリアル番号

``PROCESSOR_NAME``
  .. versionadded:: 3.10

  プロセッサ名（可読な文字列）

``PROCESSOR_DESCRIPTION``
  .. versionadded:: 3.10

  プロセッサの説明（可読な文字列）

``OS_NAME``
  .. versionadded:: 3.10

  :variable:`CMAKE_HOST_SYSTEM_NAME` 変数を参照のこと。

``OS_RELEASE``
  .. versionadded:: 3.10

  OS のサブタイプ（例えば Windows の場合は ``Professional`` など）

``OS_VERSION``
  .. versionadded:: 3.10

  OS の build ID

``OS_PLATFORM``
  .. versionadded:: 3.10

  :variable:`CMAKE_HOST_SYSTEM_PROCESSOR` 変数を参照のこと。

``MSYSTEM_PREFIX``
  .. versionadded:: 3.28

  ホストが Windows の場合にのみ利用可能。
  MSYS や MinGW の開発環境（``MSYSTEM`` という環境変数がセットされている環境）では、インストール先の Prefix で、それ以外は空の文字列。

``DISTRIB_INFO``
  .. versionadded:: 3.22

  :file:`/etc/os-release` ファイルを読み込んで、指定された ``<variable>`` を読み取って変数にセットする。

``DISTRIB_<name>``
  .. versionadded:: 3.22

  :file:`/etc/os-release` ファイルの中に ``<name>`` を変数とするエントリあればその値（`man 5 os-release`_ 参照) を取得する。

  実行例:

  .. code-block:: cmake

      cmake_host_system_information(RESULT PRETTY_NAME QUERY DISTRIB_PRETTY_NAME)
      message(STATUS "${PRETTY_NAME}")

      cmake_host_system_information(RESULT DISTRO QUERY DISTRIB_INFO)

      foreach(VAR IN LISTS DISTRO)
        message(STATUS "${VAR}=`${${VAR}}`")
      endforeach()


  出力例::

    -- Ubuntu 20.04.2 LTS
    -- DISTRO_BUG_REPORT_URL=`https://bugs.launchpad.net/ubuntu/`
    -- DISTRO_HOME_URL=`https://www.ubuntu.com/`
    -- DISTRO_ID=`ubuntu`
    -- DISTRO_ID_LIKE=`debian`
    -- DISTRO_NAME=`Ubuntu`
    -- DISTRO_PRETTY_NAME=`Ubuntu 20.04.2 LTS`
    -- DISTRO_PRIVACY_POLICY_URL=`https://www.ubuntu.com/legal/terms-and-policies/privacy-policy`
    -- DISTRO_SUPPORT_URL=`https://help.ubuntu.com/`
    -- DISTRO_UBUNTU_CODENAME=`focal`
    -- DISTRO_VERSION=`20.04.2 LTS (Focal Fossa)`
    -- DISTRO_VERSION_CODENAME=`focal`
    -- DISTRO_VERSION_ID=`20.04`

:file:`/etc/os-release` が見つからなかった場合、このコマンドはフォールバック・スクリプトを介して OS の情報を収集しようとします。
このフォールバック・スクリプトは `various distribution-specific files`_ を使って OS の識別データを収集して、それを `man 5 os-release`_ のエントリ（変数）に対応付けします。

特定の変数をフォールバックする
""""""""""""""""""""""""""""""

.. variable:: CMAKE_GET_OS_RELEASE_FALLBACK_SCRIPTS

  CMake と一緒に提供されているフォールバック・スクリプト以外にも、このリスト型の変数にユーザ自らのフォールバック・スクリプトの絶対パスを追加できます。
  このスクリプトのファイル名は次のような書式を持ちます： ``NNN-<name>.cmake`` （``NNN`` は３桁の数字で、特定の順番でスクリプトを収集した際に付与される）

.. variable:: CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_<varname>

  ユーザが提供したフォールバック・スクリプトによって収集された変数は、この命名規則に従った CMake 変数に割り当てる必要がある。
  たとえばシステムが定義した ``ID`` という変数は ``CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_ID`` に割り当てることになる。

.. variable:: CMAKE_GET_OS_RELEASE_FALLBACK_RESULT

  フォールバック・スクリプトは、割り当てた ``CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_<varname>`` を全て、このリスト型の変数に格納する必要がある。

たとえば：

.. code-block:: cmake

  # 古いディストリビューションを検出してみる
  # 参考情報：
  # - http://linuxmafia.com/faq/Admin/release-files.html
  #
  if(NOT EXISTS "${CMAKE_SYSROOT}/etc/foobar-release")
    return()
  endif()
  # 一行目の文字列だけ見る
  file(
      STRINGS "${CMAKE_SYSROOT}/etc/foobar-release" CMAKE_GET_OS_RELEASE_FALLBACK_CONTENT
      LIMIT_COUNT 1
    )
  #
  # 一行目が次の文字列の場合：
  #
  #   Foobar distribution release 1.2.3 (server) 
  #
  if(CMAKE_GET_OS_RELEASE_FALLBACK_CONTENT MATCHES "Foobar distribution release ([0-9\.]+) .*")
    set(CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_NAME Foobar)
    set(CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_PRETTY_NAME "${CMAKE_GET_OS_RELEASE_FALLBACK_CONTENT}")
    set(CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_ID foobar)
    set(CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_VERSION ${CMAKE_MATCH_1})
    set(CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_VERSION_ID ${CMAKE_MATCH_1})
    list(
        APPEND CMAKE_GET_OS_RELEASE_FALLBACK_RESULT
        CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_NAME
        CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_PRETTY_NAME
        CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_ID
        CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_VERSION
        CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_VERSION_ID
      )
  endif()
  unset(CMAKE_GET_OS_RELEASE_FALLBACK_CONTENT)


.. rubric:: Footnotes

.. [#mebibytes] 1 MiB（メビバイト）は 1024x1024 バイトと同じです。

.. _man 5 os-release: https://www.freedesktop.org/software/systemd/man/os-release.html
.. _various distribution-specific files: http://linuxmafia.com/faq/Admin/release-files.html

.. _Query Windows registry:

Windows レジストリ情報の照会
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.24

::

  cmake_host_system_information(RESULT <variable>
                                QUERY WINDOWS_REGISTRY <key> [VALUE_NAMES|SUBKEYS|VALUE <name>]
                                [VIEW (64|32|64_32|32_64|HOST|TARGET|BOTH)]
                                [SEPARATOR <separator>]
                                [ERROR_VARIABLE <result>])

ローカルのコンピュータにあるレジストリのサブキーを照会します。
レジストリ内の指定したサブキーの下に格納されているサブキーまたは値名のリスト、あるいは指定した値名のデータを返します。
エントリの照会結果は ``<variable>`` に格納されます。

.. note::

  ``CYGWIN`` のような ``Windows`` 以外のプラットフォームのレジストリを照会すると、常に空の文字列が返され、オプションの ``ERROR_VARIABLE`` に指定した変数にエラー・メッセージがセットされます。

``<key>`` にはローカル・コンピュータにあるサブキーの絶対パスを指定します。
この ``<key>`` には有効な root キーを含めて下さい。
ローカルのコンピュータで有効な root キーは次のとおりです：

* ``HKLM`` または ``HKEY_LOCAL_MACHINE``
* ``HKCU`` または ``HKEY_CURRENT_USER``
* ``HKCR`` または ``HKEY_CLASSES_ROOT``
* ``HKU`` または ``HKEY_USERS``
* ``HKCC`` または ``HKEY_CURRENT_CONFIG``

そして root  キーの下のサブキーへのパスをオプションで指定します。
このパスの区切り文字はスラッシュ（``/``）またはバックスラッシュ（``\``）を使用できます。
``<key>`` は大文字/小文字を区別しません。
たとえば：

.. code-block:: cmake

  cmake_host_system_information(RESULT result QUERY WINDOWS_REGISTRY "HKLM")
  cmake_host_system_information(RESULT result QUERY WINDOWS_REGISTRY "HKLM/SOFTWARE/Kitware")
  cmake_host_system_information(RESULT result QUERY WINDOWS_REGISTRY "HKCU\\SOFTWARE\\Kitware")

``VALUE_NAMES``
  照会時に、``<key>`` の下で定義されている値名を要素とするリストを要求する。
  もしデフォルトの値が定義されていたら、``(default)`` という特殊な名前で識別できます。

``SUBKEYS``
  照会時に、``<key>`` の下で定義されているサブキーを要素とするリストを要求する。

``VALUE <name>``
  照会時に、``<name>`` という名前を持つ値に格納されたデータを要求する。
   ``VALUE`` を指定しない、またはその引数が特殊な名前である ``(default)`` の場合、もしデフォルト値があれば、そのデータを返す。

  .. code-block:: cmake

     # HKLM/SOFTWARE/Kitware キーに対するデフォルト値を参照する
     cmake_host_system_information(RESULT result
                                   QUERY WINDOWS_REGISTRY "HKLM/SOFTWARE/Kitware")

     # 特殊なあ名前を使って HKLM/SOFTWARE/Kitware キーに対するデフォルト値を照会する
     cmake_host_system_information(RESULT result
                                   QUERY WINDOWS_REGISTRY "HKLM/SOFTWARE/Kitware"
                                   VALUE "(default)")

  サポートしている型は:

  * ``REG_SZ``
  * ``REG_EXPAND_SZ`` （返されたデータは展開される）
  * ``REG_MULTI_SZ`` （返されたデータは CMake のリスト型として展開される、 ``SEPARATOR`` というオプションも参照のこと）
  * ``REG_DWORD``
  * ``REG_QWORD``

  他の型は全て、空の文字列が返される。

``VIEW``
  どのレジストリ・ビューを照会するかを指定する。指定しない場合は、``BOTH`` ビューを使う。

  ``64``
    64bit のレジストリに照会する。``32bit Windows`` の場合は常に空の文字列を返す。

  ``32``
    32bit のレジストリに照会する。

  ``64_32``
    ``VALUE`` オプションを指定した場合、またはデフォルト値の照会の場合は ``64`` ビューを使ってレジストリを照会し、その要求が失敗したら ``32`` ビューを使ってレジストリを照会する。
    ``VALUE_NAMES`` と ``SUBKEYS`` オプションを指定した場合は、``64`` と ``32`` の両方のビューを使ってレジストリを照会し、最後に結果をマージする（重複するエントリは削除して並び替える）。

  ``32_64``
    ``VALUE`` オプションを指定した場合、またはデフォルト値の照会の場合は ``32`` ビューを使ってレジストリを照会し、その要求が失敗したら ``64`` ビューを使ってレジストリを照会する。
    ``VALUE_NAMES`` と ``SUBKEYS`` オプションを指定した場合は、``32`` と ``64`` の両方のビューを使ってレジストリを照会し、最後に結果をマージする（重複するエントリは削除して並び替える）。

  ``HOST``
    ここで指定したホストのアーキテクチャとマッチするレジストリを照会する。
    対象となるアーキテクチャは： ``64bit  Windows`` の場合は ``64``、``32bit Windows`` の場合は ``32``

  ``TARGET``
    CMake 変数の :variable:`CMAKE_SIZEOF_VOID_P` にセットしたアーキテクチャにマッチしたレジストリを照会する。
    この変数に何もセットしていなかったら ``HOST`` ビューを使って照会する。

  ``BOTH``
    ``32`` と ``64`` の両方のビューを使ってレジストリを照会する。
    照会する順番は次のルールに従う：
    CMake 変数の :variable:`CMAKE_SIZEOF_VOID_P` が定義されている場合は、この変数にセットされているアーキテクチャに応じたビューを使用する：

    * ``8``: ``64_32``
    * ``4``: ``32_64``

    CMake 変数の :variable:`CMAKE_SIZEOF_VOID_P` が定義されていない場合は、ホストのアーキテクチャに依存する：

    * ``64bit``: ``64_32``
    * ``32bit``: ``32``

``SEPARATOR``
  ``REG_MULTI_SZ`` 型の区切り文字を指定する
  指定しない場合は ``\0`` と云う文字を使う。

``ERROR_VARIABLE <result>``
  照会中に発生したエラーを返す。エラー無しで照会操作が成功した場合は、空の文字列を格納する。
