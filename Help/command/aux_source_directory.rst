aux_source_directory
--------------------

ディレクトリの中にある全てのソース・ファイルを収集する。

.. code-block:: cmake

  aux_source_directory(<dir> <variable>)

``<dir>`` の中にある全てのソース・ファイルの名前を収集し :ref:`リスト <CMake Language Lists>` にして ``<variable>`` に格納します。
このコマンドは、主にテンプレートのインスタンス化（*Template Instantiation*）を行うプロジェクト向けです。
インスタンス化したテンプレート・ファイルを ``Templates`` というサブディレクトリに保存し、この ``aux_source_directory`` コマンドを使って自動的にソース・ファイル名を収集できるので、手動でインスタンス化する必要がなくなります。

It is tempting to use this command to avoid writing the list of source files for a library or executable target.
While this seems to work, there is no way for CMake to generate a build system that knows when a new source file has been added.
Normally the generated build system knows when it needs to rerun CMake because the ``CMakeLists.txt`` file is modified to add a new source.
When the source is just added to the directory without modifying this file, one would have to manually rerun CMake to generate a build system incorporating the new file.
