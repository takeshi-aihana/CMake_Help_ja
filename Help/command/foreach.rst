foreach
-------

リスト内の要素ごとにコマンドのグループを評価する。

.. code-block:: cmake

  foreach(<loop_var> <items>)
    <commands>
  endforeach()

``<items>`` はセミコロン文字（"``;``"）または空白で区切られた要素の :ref:`リスト <CMake Language Lists>` です。
まず ``foreach`` と、これに対応する ``endforeach`` の間にある個々のコマンドは呼び出されずに記録されます。
``endforeach`` コマンドが評価された時点で、記録されていた全てのコマンドが ``<items>`` 内のアイテム毎に1回だけ呼び出されます（ループ呼び出し）。
ループの開始時に、``<loop_var>`` 変数が現在のアイテムの値にセットされます。

``<loop_var>`` 変数のスコープはループ呼び出しのスコープ（``foreach`` から ``endforeach`` の間）と同じになります。
詳細は :policy:`CMP0124` のポリシーを参照して下さい。

:command:`break` と :command:`continue` コマンドは通常のループ呼び出しから抜け出す手段を提供します。

:command:`endforeach` コマンドではオプションである ``<loop_var>`` 変数を参照できます。
この変数を使用する場合は、この ``foreach`` コマンド開始時の引数を文字通り繰り返し使用して下さい。

.. code-block:: cmake

  foreach(<loop_var> RANGE <stop>)

この呼び出しは ``<loop_var>`` を 0, 1, ..., ``<stop>`` まで正の整数単位でインクリメントしながらループを繰り返します。

.. code-block:: cmake

  foreach(<loop_var> RANGE <start> <stop> [<step>])

この呼び出しは ``<loop_var>`` を ``<start>`` から最大 ``<stop>`` まで ``<step>`` の間隔でインクリメントしながらループを繰り返します。
``<step>`` を指定しない場合は、間隔は 1 です。
この３つの引数 ``<start>`` と ``<stop>`` と ``<step>`` は全て正の整数であり、``<stop>`` は ``<start>`` よりも大きくして下さい（そうしないと、将来のリリースで変更される可能性がある仕様に知らずに抵触してしまう場合があります）。

.. code-block:: cmake

  foreach(<loop_var> IN [LISTS [<lists>]] [ITEMS [<items>]])

``<lists>`` はセミコロン文字（"``;``"）または空白で区切られた要素の :ref:`リスト <CMake Language Lists>` を格納した変数です。
この呼び出しは ``<loop_var>`` にリストの要素を順番に格納しながらループを繰り返します。
``ITEMS`` に続く ``<items>`` はリストと解釈してループを繰り返します。
``LISTS A`` と ``ITEM ${A}`` の書き方は同じ意味です。

次の例は、``LISTS`` オプションがどのように処理されるかを示します：

.. code-block:: cmake

  set(A 0;1)
  set(B 2 3)
  set(C "4 5")
  set(D 6;7 8)
  set(E "")
  foreach(X IN LISTS A B C D E)
      message(STATUS "X=${X}")
  endforeach()

の結果は次のとおりです::

  -- X=0
  -- X=1
  -- X=2
  -- X=3
  -- X=4 5
  -- X=6
  -- X=7
  -- X=8


.. code-block:: cmake

  foreach(<loop_var>... IN ZIP_LISTS <lists>)

.. versionadded:: 3.17

``<lists>`` はセミコロン文字（"``;``"）または空白で区切られた :ref:`リスト <CMake Language Lists>` 型変数の並びです。
この呼び出しは、``<loop_var>...`` に渡した変数に、次のルールに従って、``<lists>`` で対応する変数の要素を順番に格納しながらループを繰り返します：

- ``<loop_var>...`` に1個の ``<loop_var>`` が与えられたら、``<loop_var>_N`` の形式（``N`` はリスト型変数に対応する番号で、0, 1 ... として付与される）として ``<lists>`` の変数を順番に参照できる
- ``<loop_var>...`` に複数の ``<A> <B> <C>...`` が与えられたら、``<lists>`` の変数を ``<A> <B> <C>...`` の順で参照できる
- ``<loop_var>...`` の個数が ``<lists>`` の個数よりも少ない場合、足りない変数は参照できない
  
.. code-block:: cmake

  list(APPEND English one two three four)
  list(APPEND Bahasa satu dua tiga)

  foreach(num IN ZIP_LISTS English Bahasa)
      message(STATUS "num_0=${num_0}, num_1=${num_1}")
  endforeach()

  foreach(en ba IN ZIP_LISTS English Bahasa)
      message(STATUS "en=${en}, ba=${ba}")
  endforeach()

の結果は次のとおりです::

  -- num_0=one, num_1=satu
  -- num_0=two, num_1=dua
  -- num_0=three, num_1=tiga
  -- num_0=four, num_1=
  -- en=one, ba=satu
  -- en=two, ba=dua
  -- en=three, ba=tiga
  -- en=four, ba=

参考情報
^^^^^^^^

* :command:`break`
* :command:`continue`
* :command:`endforeach`
* :command:`while`
