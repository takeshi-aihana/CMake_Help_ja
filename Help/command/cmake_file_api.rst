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
  ただし、このコマンドはプロジェクト最上位にある ``CMakeLists.txt`` ファイルを読み込んで処理していく最中に呼び出されるので、（処理される前に既に設定されている） ``configureLog`` といったオブジェクトには追加できません。

  オプションごとに指定する ``<versions> ...`` の :ref:`リスト <CMake Language Lists>` には ``major`` または ``major.minor`` の形式で表したバージョン（``major`` と ``minor`` は共に整数値）を一つ以上含めるようにして下さい。
  また、このリストからはサポートされているバージョンだけを先頭から順番に選択していくので、このリストには優先的に受け入れるバージョンから並べる必要があります。
  このコマンドは CMake オブジェクトでサポートしている ``major`` バージョンよりも新しいバージョンは無視します。
  リストの中に無効なバージョンが見つかった場合、または CMake オブジェクトが指定したバージョンをサポートしていない場合はエラーで停止します。

  オプションに対応した CMake オブジェクトごとに、共有のステートレス・クエリと同等のクエリが内部的に追加されます。
  クエリのファイルは出力されません。
  このコマンドの結果は、ビルドシステムの構築時に *出力されます* 。

  クエリのファイルから、またはこの ``cmake_file_api(QUERY)`` コマンドを複数回呼び出して、同じオブジェクトに対して複数のクエリを追加しても問題はありません。
  つまり、最終的なクエリはファイル上で指定された全てのクエリ（前者）と、プロジェクトで指定されたクエリ（後者）を合わせたものになります。

サンプル
^^^^^^^^

CMake プロジェクトの中には、ビルド時にファイル API からの応答を利用して、なんらかの形式の検証タスクを実装しなければならない場合があります。
そのプロジェクトの外部にあるものに依存したクエリのファイルを作成する代わりに、この ``cmake_file_api(QUERY)`` コマンドでビルドに必要な情報を要求することができます。
そのあとに、要求した情報が常に利用できることを確認し、ビルド時に実行する独自コマンドを作成できます。

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
