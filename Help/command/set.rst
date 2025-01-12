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

  このコマンドを呼び出す前にキャッシュ・エントリが存在していない場合、または ``FORCE`` オプションを指定した場合、キャッシュ・エントリを作成して ``<value>`` をセットします。

  .. note::

    既に同じ名前を持つ通常の変数が存在している場合は、キャッシュ変数が指すキャッシュ・エントリに直接アクセスすることはできない（「:ref:`rules of variable evaluation <CMake Language Variables>`」を参照のこと）。
    :policy:`CMP0126` のポリシーが ``OLD`` にセットされている場合、現在のスコープにある通常の変数との割当が全て削除される。

  コマンドラインの :manual:`cmake(1)` で ``<type>`` を指定せずに、:option:`-D\<var\>=\<value\> <cmake -D>` オプションを使用してキャッシュ変数とキャッシュ・エントリを作成すると、型が設定されていないキャッシュ・エントリになる場合があります。
  そのような場合は、この ``set`` コマンドで追加できます。
  また、``<type>`` が ``PATH`` または ``FILEPATH`` で、``<value>`` が相対パスの場合、``set`` コマンドは現在の作業ディレクトリをベース・ディレクトリとして絶対パスに変換します。

環境変数の場合
^^^^^^^^^^^^^^

.. signature::
  set(ENV{<variable>} [<value>])
  :target: ENV

  :manual:`環境変数 <cmake-env-variables(7)>` の ``<variable>`` に ``<value>`` をセットします。
  環境変数を ``$ENV{<variable>}`` の形式で参照すると、セットした値が返ってきます。

  このコマンドの呼び出しは、現在実行中の CMake プロセスにのみ作用し、CMake の呼び出し元やプラットフォームの環境、これ以降の別のプロセス（ビルドやテスト）には影響しません。

  ``ENV{<variable>}`` の後ろに引数がない場合、または ``<value>`` が空の文字列の場合、このコマンドはその環境変数にセットされている既存の値を全てクリアします。

  ``<value>`` の後ろにある引数は全て無視します。
  余分な引数を検出したらワーニングを発行します。

参考情報
^^^^^^^^

* :command:`unset`
