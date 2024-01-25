mark_as_advanced
----------------

CMake のキャシュ変数に Advanced 属性をセットする。

.. code-block:: cmake

  mark_as_advanced([CLEAR|FORCE] <var1> ...)

名前を持つキャッシュ変数に Advanced か Advanced でないかを表す属性をセットします。

Advanced な属性の変数は ``show advanced`` オプションを ON にしない限り、:manual:`cmake-gui <cmake-gui(1)>` には表示されません。
この属性は、CMake スクリプトの中では効果はありません。

``CLEAR`` オプションを指定すると、Advanced な属性を持つ変数は Advanced ではない属性に戻ります。
``FORCE`` オプションを指定すると、強制的に Advanced な属性になります。
``FORCE`` や ``CLEAR`` のいずれのオプションを指定しない場合、新規に作成した変数は Advanced 属性になりますが、既存の変数の属性は変更されません。

.. versionchanged:: 3.17
  このコマンドに渡された変数が、未だキャッシュに存在していなかったら無視する（詳細は :policy:`CMP0102` ポリシーを参照のこと）。
