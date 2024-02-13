add_dependencies
----------------

プロジェクト最上位のターゲット間にある依存関係を追加する。

.. code-block:: cmake

  add_dependencies(<target> [<target-dependency>]...)

プロジェクトの最上位にある任意の ``<target>`` に、同じ最上位にある別のターゲットとの依存を追加し、``<target>`` を実行する前にそれらが確実にビルドされるよう CMake に指示します。
最上位にある ``<target>`` は :command:`add_executable` や :command:`add_library` や :command:`add_custom_target` コマンドのいずれかを使って追加されたターゲットです（ただし ``install`` のような CMake が追加するターゲットではありません）。

このコマンドは、``<target>`` 自身はビルドしないので、「:ref:`インポートしたターゲット <Imported Targets>`」や「:ref:`インタフェース・ライブラリ <Interface Libraries>` 」に追加された依存関係は、その最上位ディレクトリから追跡されます。

.. versionadded:: 3.3
  インタフェース・ライブラリへの依存関係の追加をサポートした。

参考情報
^^^^^^^^

* 独自のルールに「ファイル単位」の依存関係を追加する :command:`add_custom_target` や :command:`add_custom_command` コマンドの ``DEPENDS`` オプション。

* :prop_sf:`OBJECT_DEPENDS` というソース・ファイルのプロパティは「ファイル単位」の依存関係をオブジェクト・ファイルに追加する。
