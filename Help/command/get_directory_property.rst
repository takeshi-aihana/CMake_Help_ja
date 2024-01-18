get_directory_property
----------------------

``DIRECTORY`` スコープのプロパティを取得する。

.. code-block:: cmake

  get_directory_property(<variable> [DIRECTORY <dir>] <prop-name>)

取得したディレクトリ・スコープのプロパティを ``<variable>`` という変数に格納します。

``DIRECTORY`` オプションには、現在のディレクトリの代わりにプロパティを取得するディレクトリを指定します。
相対パスを指定すると、現在のディレクトリに対する相対ディレクトリとして扱います。
CMake は :command:`add_subdirectory` コマンドを呼び出してディレクトリを追加するか、またはプロジェクト最上位のディレクトリを起点として、指定したディレクトリを認識している必要があります。

.. versionadded:: 3.19
  ``<dir>`` に :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できるようになった。

指定したディレクトリ・スコープでプロパティが何も定義されていない場合、空の文字列が格納されます。
``INHERITED`` プロパティを指定してディレクトリ・スコープからプロパティが見つからなかったら、:command:`define_property` コマンドで説明したように、親スコープからプロパティを見つけます。

.. code-block:: cmake

  get_directory_property(<variable> [DIRECTORY <dir>]
                         DEFINITION <var-name>)

ディレクトリから変数の定義を取得します。
この呼び出し方は別のディレクトリから変数の定義を取得する際に便利です。

参考情報
^^^^^^^^

* :command:`define_property`
* さらに汎用的な :command:`get_property` コマンド
