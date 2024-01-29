set_directory_properties
------------------------

現在のディレクトリとそのサブディレクトリのプロパティをセットする。

.. code-block:: cmake

  set_directory_properties(PROPERTIES prop1 value1 [prop2 value2] ...)

現在のディレクトリとそのサブディレクトリに key-value ペア形式で構成されるプロパティ（``prop1 value1 prop2 value2 ...``）をセットします。

:command:`set_property(DIRECTORY)` コマンドも参照して下さい。

CMake で認識しているプロパティの一覧と、各プロパティの作用については「:ref:`Directory Properties`」を参照して下さい。

参考情報
^^^^^^^^

* :command:`define_property`
* :command:`get_directory_property`
* より汎用的な :command:`set_property` コマンド
