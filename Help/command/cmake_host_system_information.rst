cmake_host_system_information
-----------------------------

いろいろなホスト・システムの情報を照会する。

概要
^^^^

.. parsed-literal::

  `Query host system specific information`_
    cmake_host_system_information(RESULT <variable> QUERY <key> ...)

  `Query Windows registry`_
    cmake_host_system_information(RESULT <variable> QUERY WINDOWS_REGISTRY <key> ...)

.. _Query host system specific information:

ホスト・システム固有の情報の照会
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

インタフェース変数のフォールバック
""""""""""""""""""""""""""""""""""

.. variable:: CMAKE_GET_OS_RELEASE_FALLBACK_SCRIPTS

  CMake と一緒に提供されているスクリプトの他、このリスト型の変数にユーザ自らのスクリプトの絶対パスを追加できます。
  このスクリプトのファイル名は次のような書式を持ちます：``NNN-<name>.cmake``（`NNN`` は３桁の数字で、特定の順番でスクリプトを収集した際に付与される）

.. variable:: CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_<varname>

  Variables collected by the user provided fallback script ought to be assigned to CMake variables using this naming convention.
  Example, the ``ID`` variable from the manual becomes ``CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_ID``.

.. variable:: CMAKE_GET_OS_RELEASE_FALLBACK_RESULT

  The fallback script ought to store names of all assigned  ``CMAKE_GET_OS_RELEASE_FALLBACK_RESULT_<varname>`` variables in this list.

Example:

.. code-block:: cmake

  # Try to detect some old distribution
  # See also
  # - http://linuxmafia.com/faq/Admin/release-files.html
  #
  if(NOT EXISTS "${CMAKE_SYSROOT}/etc/foobar-release")
    return()
  endif()
  # Get the first string only
  file(
      STRINGS "${CMAKE_SYSROOT}/etc/foobar-release" CMAKE_GET_OS_RELEASE_FALLBACK_CONTENT
      LIMIT_COUNT 1
    )
  #
  # Example:
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

.. [#mebibytes] One MiB (mebibyte) is equal to 1024x1024 bytes.

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

Performs query operations on local computer registry subkey.
Returns a list of subkeys or value names that are located under the specified subkey in the registry or the data of the specified value name.
The result of the queried entity is stored in ``<variable>``.

.. note::

  Querying registry for any other platforms than ``Windows``, including  ``CYGWIN``, will always returns an empty string and sets an error message in the variable specified with sub-option ``ERROR_VARIABLE``.

``<key>`` specify the full path of a subkey on the local computer.
The `<key>`` must include a valid root key. Valid root keys for the local computer are:

* ``HKLM`` or ``HKEY_LOCAL_MACHINE``
* ``HKCU`` or ``HKEY_CURRENT_USER``
* ``HKCR`` or ``HKEY_CLASSES_ROOT``
* ``HKU`` or ``HKEY_USERS``
* ``HKCC`` or ``HKEY_CURRENT_CONFIG``

And, optionally, the path to a subkey under the specified root key. The path separator can be the slash or the backslash.
``<key>`` is not case sensitive.
For example:

.. code-block:: cmake

  cmake_host_system_information(RESULT result QUERY WINDOWS_REGISTRY "HKLM")
  cmake_host_system_information(RESULT result QUERY WINDOWS_REGISTRY "HKLM/SOFTWARE/Kitware")
  cmake_host_system_information(RESULT result QUERY WINDOWS_REGISTRY "HKCU\\SOFTWARE\\Kitware")

``VALUE_NAMES``
  Request the list of value names defined under ``<key>``.
  If a default value is defined, it will be identified with the special name ``(default)``.

``SUBKEYS``
  Request the list of subkeys defined under ``<key>``.

``VALUE <name>``
  Request the data stored in value named ``<name>``. If ``VALUE`` is not specified or argument is the special name ``(default)``, the content of the default value, if any, will be returned.

  .. code-block:: cmake

     # query default value for HKLM/SOFTWARE/Kitware key
     cmake_host_system_information(RESULT result
                                   QUERY WINDOWS_REGISTRY "HKLM/SOFTWARE/Kitware")

     # query default value for HKLM/SOFTWARE/Kitware key using special value name
     cmake_host_system_information(RESULT result
                                   QUERY WINDOWS_REGISTRY "HKLM/SOFTWARE/Kitware"
                                   VALUE "(default)")

  Supported types are:

  * ``REG_SZ``.
  * ``REG_EXPAND_SZ``. The returned data is expanded.
  * ``REG_MULTI_SZ``. The returned is expressed as a CMake list. See also
    ``SEPARATOR`` sub-option.
  * ``REG_DWORD``.
  * ``REG_QWORD``.

  For all other types, an empty string is returned.

``VIEW``
  Specify which registry views must be queried. When not specified,
  ``BOTH`` view is used.

  ``64``
    Query the 64bit registry. On ``32bit Windows``, returns always an empty  string.

  ``32``
    Query the 32bit registry.

  ``64_32``
    For ``VALUE`` sub-option or default value, query the registry using view ``64``, and if the request failed, query the registry using view ``32``.
    For ``VALUE_NAMES`` and ``SUBKEYS`` sub-options, query both views (``64`` and ``32``) and merge the results (sorted and duplicates removed).

  ``32_64``
    For ``VALUE`` sub-option or default value, query the registry using view ``32``, and if the request failed, query the registry using view ``64``.
    For ``VALUE_NAMES`` and ``SUBKEYS`` sub-options, query both views (``32`` and ``64``) and merge the results (sorted and duplicates removed).

  ``HOST``
    Query the registry matching the architecture of the host: ``64`` on ``64bit  Windows`` and ``32`` on ``32bit Windows``.

  ``TARGET``
    Query the registry matching the architecture specified by :variable:`CMAKE_SIZEOF_VOID_P` variable.
    If not defined, fallback to ``HOST`` view.

  ``BOTH``
    Query both views (``32`` and ``64``).
    The order depends of the following rules:
    If :variable:`CMAKE_SIZEOF_VOID_P` variable is defined. Use the following view depending of the content of this variable:

    * ``8``: ``64_32``
    * ``4``: ``32_64``

    If :variable:`CMAKE_SIZEOF_VOID_P` variable is not defined, rely on architecture of the host:

    * ``64bit``: ``64_32``
    * ``32bit``: ``32``

``SEPARATOR``
  Specify the separator character for ``REG_MULTI_SZ`` type. When not pecified, the character ``\0`` is used.

``ERROR_VARIABLE <result>``
  Returns any error raised during query operation. In case of success, the  variable holds an empty string.
