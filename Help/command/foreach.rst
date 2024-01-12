foreach
-------

リスト内の要素ごとにコマンドのグループを評価する。

.. code-block:: cmake

  foreach(<loop_var> <items>)
    <commands>
  endforeach()

``<items>`` はセミコロン文字（"``;``"）または空白で区切られた要素の :ref:`リスト <CMake Language Lists>` です。
``foreach`` と、これに対応する ``endforeach`` の間にある全てのコマンドは呼び出されずに記録されます。
``endforeach`` コマンドが評価された時点で、記録されていた一連のコマンドが ``<items>`` 内のアイテム毎に1回だけ呼び出されます（ループ呼び出し）。
ループの開始時に、``<loop_var>`` 変数が現在のアイテムの値にセットされます。

``<loop_var>`` 変数のスコープはループ呼び出しのスコープと同じになります。
詳細は :policy:`CMP0124` のポリシーを参照して下さい。

:command:`break` と :command:`continue` コマンドは通常のループ呼び出しから抜け出す手段を提供します。

command:`endforeach` コマンドではオプションである ``<loop_var>`` 変数を参照できます。
この変数を使用する際は、この ``foreach`` コマンドの開始時の引数をそのまま繰り返していく必要があります。

.. code-block:: cmake

  foreach(<loop_var> RANGE <stop>)

In this variant, ``foreach`` iterates over the numbers 0, 1, ... up to (and including) the nonnegative integer ``<stop>``.

.. code-block:: cmake

  foreach(<loop_var> RANGE <start> <stop> [<step>])

In this variant, ``foreach`` iterates over the numbers from ``<start>`` up to at most ``<stop>`` in steps of ``<step>``.
If ``<step>`` is not specified, then the step size is 1.
The three arguments ``<start>`` ``<stop>`` ``<step>`` must all be nonnegative integers, and ``<stop>`` must not be smaller than ``<start>``; otherwise you enter the danger zone of undocumented behavior that may change in future releases.

.. code-block:: cmake

  foreach(<loop_var> IN [LISTS [<lists>]] [ITEMS [<items>]])

In this variant, ``<lists>`` is a whitespace or semicolon separated list of list-valued variables.
The ``foreach`` command iterates over each item in each given list.
The ``<items>`` following the ``ITEMS`` keyword are processed as in the first variant of the ``foreach`` command.
The forms ``LISTS A`` and ``ITEMS ${A}`` are equivalent.

The following example shows how the ``LISTS`` option is processed:

.. code-block:: cmake

  set(A 0;1)
  set(B 2 3)
  set(C "4 5")
  set(D 6;7 8)
  set(E "")
  foreach(X IN LISTS A B C D E)
      message(STATUS "X=${X}")
  endforeach()

yields::

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

In this variant, ``<lists>`` is a whitespace or semicolon separated list of list-valued variables.
The ``foreach`` command iterates over each list simultaneously setting the iteration variables as follows:

- if the only ``loop_var`` given, then it sets a series of ``loop_var_N`` variables to the current item from the corresponding list;
- if multiple variable names passed, their count should match the lists variables count;
- if any of the lists are shorter, the corresponding iteration variable is not defined for the current iteration.

.. code-block:: cmake

  list(APPEND English one two three four)
  list(APPEND Bahasa satu dua tiga)

  foreach(num IN ZIP_LISTS English Bahasa)
      message(STATUS "num_0=${num_0}, num_1=${num_1}")
  endforeach()

  foreach(en ba IN ZIP_LISTS English Bahasa)
      message(STATUS "en=${en}, ba=${ba}")
  endforeach()

yields::

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
