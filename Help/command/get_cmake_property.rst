get_cmake_property
------------------

CMake インスタンスからグローバルなプロパティを取得する。

.. code-block:: cmake

  get_cmake_property(<var> <property>)

CMake のインスタンスからグローバルなプロパティを一つ取得します。
取得した ``<property>`` の値が ``<var>`` という変数に格納されます。
プロパティが見つからなかったら、``<var>`` には ``NOTFOUND`` が格納されます。
ここで指定できるプロパティについては :manual:`cmake-properties(7)` を参照して下さい。

このコマンドは、グローバルなプロパティに加えて（歴史的な理由により） :prop_dir:`VARIABLES` や :prop_dir:`MACROS` も取得できます。
また :command:`install` コマンドに指定したコンポーネントの :ref:`リスト <CMake Language Lists>` である特殊な ``COMPONENTS`` プロパティも取得できます。

参考情報
^^^^^^^^

* :command:`get_property` コマンドの ``GLOBAL`` オプション
