cmake_file_api
--------------

.. versionadded:: 3.27

:manual:`CMake のファイル API <cmake-file-api(7)>` を使う。

.. signature::
  cmake_file_api(QUERY ...)

  ``QUERY`` サブコマンドで、現在の CMake 呼び出しに対するファイル API を使ったクエリを追加します。

  .. code-block:: cmake

    cmake_file_api(
      QUERY
      API_VERSION <version>
      [CODEMODEL <versions>...]
      [CACHE <versions>...]
      [CMAKEFILES <versions>...]
      [TOOLCHAINS <versions>...]
    )

  ``API_VERSION`` オプションは必須です。
  このバージョンの CMake でサポートしている ``<version>`` の値は 1 のみ。
  クエリの返り値とファイルのパスについて詳細は :ref:`file-api v1` を参照して下さい。

  ``CODEMODEL`` や ``CACHE`` や  ``CMAKEFILES`` や  ``TOOLCHAINS`` といったオプションはそれぞれプロジェクトが要求できる「CMake オブジェクト」に対応します。
  ただし、このコマンドはプロジェクト最上位にある ``CMakeLists.txt`` ファイルの読み込みを開始する前に、呼び出しておく必要があるため、たとえば ``configureLog`` というオブジェクトの類は設定できません。

  For each of the optional keywords, the ``<versions>`` list must contain one or more version values of the form ``major`` or ``major.minor``, where ``major`` and ``minor`` are integers.
  Projects should list the versions they accept in their preferred order, as only the first supported value from the list will be selected.
  The command will ignore versions with a ``major``  version higher than any major version it supports for that object kind.
  It will raise an error if it encounters an invalid version number, or if none of the requested versions is supported.

  For each type of object kind requested, a query equivalent to a shared, stateless query will be added internally.
  No query file will be created in the file system.
  The reply *will* be written to the file system at generation time.

  It is not an error to add a query for the same thing more than once, whether from query files or from multiple calls to ``cmake_file_api(QUERY)``.
  The final set of queries will be a merged combination of all queries specified on disk and queries submitted by the project.

サンプル
^^^^^^^^

A project may want to use replies from the file API at build time to implement some form of verification task.
Instead of relying on something outside of CMake to create a query file, the project can use ``cmake_file_api(QUERY)`` to request the required information for the current run.
It can then create a custom command to run at build time, knowing that the requested information should always be available.

.. code-block:: cmake

  cmake_file_api(
    QUERY
    API_VERSION 1
    CODEMODEL 2.3
    TOOLCHAINS 1
  )

  add_custom_target(verify_project
    COMMAND ${CMAKE_COMMAND}
      -D BUILD_DIR=${CMAKE_BINARY_DIR}
      -D CONFIG=$<CONFIG>
      -P ${CMAKE_CURRENT_SOURCE_DIR}/verify_project.cmake
  )
