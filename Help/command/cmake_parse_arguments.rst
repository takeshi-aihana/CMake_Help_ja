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
マクロや関数に渡された引数をオプションとして処理し、オプションの値を保持する変数をそれぞれ定義します。

まず最初に、``<args>...`` に渡された引数を処理します。
これは :command:`macro` や :command:`function` の中で参照できます。

.. versionadded:: 3.7
  ``PARSE_ARGV`` というシグネチャは :command:`function` の中でしか使用できません。
  その場合、解析する引数は関数を呼び出した際の ``ARGV#`` という変数から受け取ります。
  解析は ``<N>`` 番目の引数から始まります（``<N>`` は符号なし整数）。
  これにより ``;`` のような特殊な文字を引数の中に含めることができます。

``<options>`` には、マクロや関数に渡されるオプションのうち追加で値を必要としない全てのキーワードが含まれています。
たとえば、:command:`install` コマンドのオプションで ``OPTIONAL`` というキーワードが該当します。

同様に ``<one_value_keywords>`` には、マクロや関数に渡されるオプションのうち追加で値を一つだけ受け取る全てのキーワードが含まれています。
たとえば、:command:`install` コマンドのオプションで ``DESTINATION`` というキーワードが該当します。

また ``<multi_value_keywords>`` には、マクロや関数に渡されるオプションのうち追加で複数の値を受け取る全てのキーワードが含まれています。
たとえば、:command:`install` コマンドのオプションで ``TARGETS`` や ``FILES`` というキーワードが該当します。

.. versionchanged:: 3.5
  全てのキーワードがユニークな扱いになりました。
  つまり、全てのキーワードは ``<options>``、``<one_value_keywords>`` 、または ``<multi_value_keywords>`` のいずれかとし、それを一回だけ指定できます。
  キーワードを重複して指定すると警告が出力されます。

キーワードの受け取りが完了すると ``cmake_parse_arguments`` は、変数の ``<options>`` や ``<one_value_keywords>`` や ``<multi_value_keywords>`` に格納されたキーワードに対して、それぞれ指定した ``<prefix>`` の後ろにアンダースコア（``"_"``）とキーワードの名前が続く内部変数を作成します。

これらの変数はオプション列のそれぞれの値を格納するか、関連するオプションが無ければ未定義になります。
特定の値を持たない ``<options>`` の場合は、オプションが含まれているかどうかに関係なく、常に ``TRUE`` または ``FALSE`` になります。

これ以外の残りの引数は、全て ``<prefix>_UNPARSED_ARGUMENTS`` という変数に格納され、その中の引数がすべて解析されたら未定義になります。
この変数をあとでチェックして、マクロが認識できない引数が渡されたかどうかを確認できます。

.. versionadded:: 3.15
   ``<one_value_keywords>`` と ``<multi_value_keywords>`` に含まれるキーワードが値をまったく受け取らなかった場合、それらは変数の ``<prefix>_KEYWORDS_MISSING_VALUES`` の中に格納され、その中にある全てのキーワードが値を受け取ると未定義になります。
   この変数をあとでチェックして、値を受け取っていないキーワードの存在を確認できます。

たとえば、次のようなマクロをみてみましょう。
この ``my_install()`` というマクロは、:command:`install` コマンドと同じような引数を受け取ります：

.. code-block:: cmake

   macro(my_install)
       set(options OPTIONAL FAST)
       set(oneValueArgs DESTINATION RENAME)
       set(multiValueArgs TARGETS CONFIGURATIONS)
       cmake_parse_arguments(MY_INSTALL "${options}" "${oneValueArgs}"
                             "${multiValueArgs}" ${ARGN} )

       # ...

この ``my_install()`` が次のように呼び出されたとすると：

.. code-block:: cmake

   my_install(TARGETS foo bar DESTINATION bin OPTIONAL blub CONFIGURATIONS)

マクロの中で ``cmake_parse_arguments`` コマンドが呼び出されると、次の変数をセットまたは未定義にします::

   MY_INSTALL_OPTIONAL = TRUE
   MY_INSTALL_FAST = FALSE # このオプションは my_install() の呼び出しでは指定されなかった
   MY_INSTALL_DESTINATION = "bin"
   MY_INSTALL_RENAME <UNDEFINED> # このオプションも指定されなかった
   MY_INSTALL_TARGETS = "foo;bar"
   MY_INSTALL_CONFIGURATIONS <UNDEFINED> # このオプションは指定されたが値を受け取らなかった
   MY_INSTALL_UNPARSED_ARGUMENTS = "blub" # "OPTIONAL" は値を受け取らないはず
   MY_INSTALL_KEYWORDS_MISSING_VALUES = "CONFIGURATIONS"
            # "CONFIGURATIONS" は値を受け取るはずが、何も受け取らなかった

このあと、それらの変数を継続して処理できます。

Keywords terminate lists of values,（訳注：意味不明）

``one_value_keyword`` の処理中に、別に認識されたキーワードが続く場合は、これを新しいオプションの始まりと解釈します。
たとえば ``my_install(TARGETS foo DESTINATION OPTIONAL)`` というマクロの呼び出しを処理すると、``MY_INSTALL_DESTINATION`` という変数にはじめは ``"OPTIONAL"`` がセットされますが、``OPTIONAL`` はキーワードであるので、この ``MY_INSTALL_DESTINATION`` は空になり（さらに、``DESTINATION`` というキーワードが ``MY_INSTALL_KEYWORDS_MISSING_VALUES`` に追加され）、最終的に ``MY_INSTALL_OPTIONAL`` という変数には ``TRUE`` がセットされます。

参考情報
^^^^^^^^

* :command:`function`
* :command:`macro`
