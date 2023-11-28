cmake_parse_arguments
---------------------

関数またはマクロの引数を解析する。

.. code-block:: cmake

  cmake_parse_arguments(<prefix> <options> <one_value_keywords>
                        <multi_value_keywords> <args>...)

  cmake_parse_arguments(PARSE_ARGV <N> <prefix> <options>
                        <one_value_keywords> <multi_value_keywords>)

.. versionadded:: 3.5
  このコマンドはネィティブな実装になりました。
  以前のバージョンでは、:module:`CMakeParseArguments` というモジュールで定義していました。

このコマンドはマクロや関数の中で使用します。
マクロや関数に渡された引数をオプションとして処理し、それぞれのオプションの値を保持する変数の集合を定義します。

まず最初に、``<args>...`` に渡された引数を処理します。
これは :command:`macro` や :command:`function` のいずれかで使用できます。

.. versionadded:: 3.7
  ``PARSE_ARGV`` というシグネチャは :command:`function` の中でしか使用できません。
  その場合、解析する引数は関数を呼び出した際の ``ARGV#`` という変数から受け取ります。
  解析は ``<N>`` 番目の引数から始まります（``<N>`` は符号なし整数）。
  これにより ``;`` のような特殊な文字を引数の中に含めることができます。

``<options>`` には、マクロに渡されるオプションのうち追加で値を指定しない全てのキーワードが含まれています：
たとえば、:command:`install` コマンドのオプションで ``OPTIONAL`` というキーワードが該当します。

同様に ``<one_value_keywords>`` には、マクロに渡されるオプションのうち追加で値を一つだけ受け取る全てのキーワードが含まれています：
たとえば、:command:`install` コマンドのオプションで ``DESTINATION`` というキーワードが該当します。

また ``<multi_value_keywords>`` には、マクロに渡されるオプションのうち追加で複数の値を受け取る全てのキーワードが含まれています：
たとえば、:command:`install` コマンドのオプションで ``TARGETS`` や ``FILES`` というキーワードが該当します。

.. versionchanged:: 3.5
  全てのキーワードがユニークな扱いになりました。
  つまり、全てのキーワードは ``<options>``、``<one_value_keywords>`` 、または ``<multi_value_keywords>`` のいずれかで、それを一回だけ指定できます。
  キーワードを重複して指定すると警告が出力されます。

キーワードの受け取りが完了すると ``cmake_parse_arguments`` は、変数の ``<options>`` や ``<one_value_keywords>`` や ``<multi_value_keywords>`` に格納されたキーワードに対して、それぞれ指定した ``<prefix>`` の後ろにアンダースコア（``"_"``）とキーワードの名前が続く内部変数を作成します。

これらの変数はオプション列のそれぞれの値を格納するか、関連するオプションが無ければ未定義になります。
値を保たない ``<options>`` の場合は、オプションが含まれているかどうかに関係なく、常に ``TRUE`` または ``FALSE`` に定義される。

これ以外の残りの引数は、すべて ``<prefix>_UNPARSED_ARGUMENTS`` という変数に格納され、その中の引数がすべて解析されたら未定義になります。
この変数をあとでチェックして、マクロが認識できない引数が渡されたかどうかを確認できます。

.. versionadded:: 3.15
   ``<one_value_keywords>`` and ``<multi_value_keywords>`` that were given no values at all are collected in a variable ``<prefix>_KEYWORDS_MISSING_VALUES`` that will be undefined if all keywords received values.
   This can be checked to see if there were keywords without any values given.

Consider the following example macro, ``my_install()``, which takes similar arguments to the real :command:`install` command:

.. code-block:: cmake

   macro(my_install)
       set(options OPTIONAL FAST)
       set(oneValueArgs DESTINATION RENAME)
       set(multiValueArgs TARGETS CONFIGURATIONS)
       cmake_parse_arguments(MY_INSTALL "${options}" "${oneValueArgs}"
                             "${multiValueArgs}" ${ARGN} )

       # ...

Assume ``my_install()`` has been called like this:

.. code-block:: cmake

   my_install(TARGETS foo bar DESTINATION bin OPTIONAL blub CONFIGURATIONS)

After the ``cmake_parse_arguments`` call the macro will have set or undefined the following variables::

   MY_INSTALL_OPTIONAL = TRUE
   MY_INSTALL_FAST = FALSE # was not used in call to my_install
   MY_INSTALL_DESTINATION = "bin"
   MY_INSTALL_RENAME <UNDEFINED> # was not used
   MY_INSTALL_TARGETS = "foo;bar"
   MY_INSTALL_CONFIGURATIONS <UNDEFINED> # was not used
   MY_INSTALL_UNPARSED_ARGUMENTS = "blub" # nothing expected after "OPTIONAL"
   MY_INSTALL_KEYWORDS_MISSING_VALUES = "CONFIGURATIONS"
            # No value for "CONFIGURATIONS" given

You can then continue and process these variables.

Keywords terminate lists of values, e.g. if directly after a ``one_value_keyword`` another recognized keyword follows, this is interpreted as the beginning of the new option.
E.g. ``my_install(TARGETS foo DESTINATION OPTIONAL)`` would result in ``MY_INSTALL_DESTINATION`` set to ``"OPTIONAL"``, but as ``OPTIONAL`` is a keyword itself ``MY_INSTALL_DESTINATION`` will be empty (but added to ``MY_INSTALL_KEYWORDS_MISSING_VALUES``) and ``MY_INSTALL_OPTIONAL`` will therefore be set to ``TRUE``.

参考情報
^^^^^^^^

* :command:`function`
* :command:`macro`
