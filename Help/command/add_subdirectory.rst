add_subdirectory
----------------

プロジェクトにサブディレクトリを追加する。

.. code-block:: cmake

  add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL] [SYSTEM])

プロジェクトのビルド・ツリーにサブディレクトリを追加します。
``source_dir`` には ``CMakeLists.txt`` とソース・ファイルがあるディレクトリを指定します。
``source_dir`` が相対パスの場合は :variable:`CMAKE_CURRENT_SOURCE_DIR` をベースディレクトリにして評価します。
``binary_dir`` にはファイルを出力するディレクトリを指定します。
``binary_dir`` が相対パスの場合は :variable:`CMAKE_CURRENT_BINARY_DIR` をベース・ディレクトリにして評価します。
``binary_dir`` を指定しない場合は、（相対パスを評価する前の） ``source_dir`` のディレクトリを使用します。
``source_dir`` にある ``CMakeLists.txt`` は、後続のコマンドで入力ファイルを処理する前に、CMake によって直ちに実行されます。

If the ``EXCLUDE_FROM_ALL`` argument is provided then targets in the subdirectory will not be included in the ``ALL`` target of the parent directory by default, and will be excluded from IDE project files.
Users must explicitly build targets in the subdirectory.
This is meant for use when the subdirectory contains a separate part of the project that is useful but not necessary, such as a set of examples.
Typically the subdirectory should contain its own :command:`project` command invocation so that a full build system will be generated in the subdirectory (such as a Visual Studio IDE solution file).
Note that inter-target dependencies supersede this exclusion.
If a target built by the parent project depends on a target in the subdirectory, the dependee target will be included in the parent project build system to satisfy the dependency.

.. versionadded:: 3.25
  If the ``SYSTEM`` argument is provided, the :prop_dir:`SYSTEM` directory property of the subdirectory will be set to true.
  This property is used to initialize the :prop_tgt:`SYSTEM` property of each non-imported target created in that subdirectory.
