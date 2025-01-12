option
------

ユーザが任意に選択できる論理型のオプション変数を定義する。

.. code-block:: cmake

  option(<variable> "<help_text>" [value])

``<value>`` の初期値が定義されていない場合は、論理型の ``OFF`` がデフォルト値になります。
``<variable>`` が既に通常の変数、またはキャッシュ変数として定義されている場合、このコマンドは何も実行しません（:policy:`CMP0077` のポリシーを参照のこと）。

他のオプション値に依存するオプションについては  :module:`CMakeDependentOption` モジュールのヘルプを参照して下さい。

CMake でプロジェクトをビルドする際は、このオプションの値を使用して論理型のキャッシュ変数を作成します。
CMake でスクリプトを実行する際は、論理型の変数にこのオプションの値が格納されます。
