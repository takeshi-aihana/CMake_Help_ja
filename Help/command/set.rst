set
---

通常の変数やキャッシュ変数や環境変数に特定の値をセットする。
（通常の変数とキャッシュ変数の間にある作用の違いやそれぞれのスコープについては「:ref:`CMake のいろいろな変数 <CMake Language Variables>`」を参照のこと）。

``<value> ...`` を指定する場合は、0個以上の引数が必要である
（この複数の引数は「:ref:`セミコロンで区切られたリスト <CMake Language Lists>` 」として、それぞれセットする値が結合されているものとする）。

通常の変数の場合
^^^^^^^^^^^^^^^^

.. signature::
  set(<variable> <value>... [PARENT_SCOPE])
  :target: normal

  現在の関数またはディレクトリのスコープの中で、``<variable>`` に値をセットしたり解除します：

  * ``<value>`` が与えられたら ``<variable>`` に ``<value>`` をセットする。
  * ``<value>`` が与えられなかったら ``<variable>`` の値を解除する（これは :command:`unset(<variable>) <unset>` コマンドの呼び出しと同じ）。

  ``PARENT_SCOPE`` オプションを指定すると、``<variable>`` は現在のスコープよりも上のスコープの中で値をセットします。
  新しいディレクトリを作成する、または :command:`function` コマンドを呼び出す度に、新しいスコープを生成します。
  スコープは  :command:`block` コマンドを使って生成することもできます。
  ``set(PARENT_SCOPE)`` の呼び出しは、親ディレクトリや、関数の呼び出し元、包含するブロックなどそれぞれ対応するスコープに対して値をセットすることを意味します。
  ``<variable>`` にセットした値の前の状態は、現在のスコープのままです（例えば、変数の一つ前が未定義だった場合のスコープは未定義であり、値がセットされていたらその値のままです）。

  :command:`block(PROPAGATE)` と :command:`return(PROPAGATE)` コマンドは、親のスコープに対して呼び出す :command:`set(PARENT_SCOPE)` と :command:`unset(PARENT_SCOPE)` の代替として使用できます。

.. include:: UNSET_NOTE.txt

キャッシュ変数の場合
^^^^^^^^^^^^^^^^^^^^

.. signature::
  set(<variable> <value>... CACHE <type> <docstring> [FORCE])
  :target: CACHE

  キャッシュ変数の ``<variable>`` に値をセットします（**キャッシュ・エントリ**）。
  キャッシュ変数は、ユーザが選択が可能な設定値をプロジェクトに提供することを目的としているため、**デフォルトではキャッシュ・エントリを上書きしません**。
  従って既存のキャッシュ・エントリを上書きする場合は ``FORCE`` オプションを使います。

  キャッシュ・エントリの型を表す ``<type>`` オプションには、以下のいずれかを指定して下さい：

    ``BOOL``
      論理値の ``ON/OFF`` をセットする。
      :manual:`cmake-gui(1)` の場合はチェックボックスを使う。

    ``FILEPATH``
      ローカルのファイルのパス名をセットする。
      :manual:`cmake-gui(1)` の場合はファイル・ダイアログを使う。

    ``PATH``
      ローカルのディレクトリのパス名をセットとする。
      :manual:`cmake-gui(1)` の場合はファイル・ダイアログを使う。

    ``STRING``
      文字列をセットする。
      :manual:`cmake-gui(1)` の場合で :prop_cache:`STRINGS` 型のキャッシュ・エントリのプロパティがセットされている場合は、テキスト・フィールドやドロップダウン・セレクタを使う。
      
    ``INTERNAL``
      文字列をセットする。
      :manual:`cmake-gui(1)` の場合は、この型のキャッシュ・エントリは表示しない。
      これは CMake を実行する度に変数を永続的に保存する際に使用する。
      この型を指定した場合は ``FORCE`` オプションを追加すること。

  ``<docstring>`` には、:manual:`cmake-gui(1)` のユーザに提示するオプションのサマリを文字列として指定して下さい。

  このコマンドを呼び出す前にキャッシュ・エントリが存在していない場合、または ``FORCE`` オプションを指定した場合、キャッシュ・エントリを作成して ``<value>`` をセットする。

  .. note::

    The content of the cache variable will not be directly accessible if a normal variable of the same name already exists (see :ref:`rules of variable evaluation <CMake Language Variables>`).
    If policy :policy:`CMP0126` is set to ``OLD``, any normal variable binding in the current scope will be removed.

  It is possible for the cache entry to exist prior to the call but have no type set if it was created on the :manual:`cmake(1)` command line by a user through the :option:`-D\<var\>=\<value\> <cmake -D>` option without specifying a type.
  In this case the ``set`` command will add the type.
  Furthermore, if the ``<type>`` is ``PATH`` or ``FILEPATH`` and the ``<value>`` provided on the command line is a relative path, then the ``set`` command will treat the path as relative to the current working directory and convert it to an absolute path.

環境変数の場合
^^^^^^^^^^^^^^

.. signature::
  set(ENV{<variable>} [<value>])
  :target: ENV

  Sets an :manual:`Environment Variable <cmake-env-variables(7)>` to the given value.
  Subsequent calls of ``$ENV{<variable>}`` will return this new value.

  This command affects only the current CMake process, not the process from which CMake was called, nor the system environment at large, nor the environment of subsequent build or test processes.

  If no argument is given after ``ENV{<variable>}`` or if ``<value>`` is an empty string, then this command will clear any existing value of the environment variable.

  Arguments after ``<value>`` are ignored. If extra arguments are found, then an author warning is issued.

参考情報
^^^^^^^^

* :command:`unset`
