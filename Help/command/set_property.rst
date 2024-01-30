set_property
------------

指定したスコープ内で有効な名前付きのプロパティをオブジェクトにセットする。

.. code-block:: cmake

  set_property(<GLOBAL                      |
                DIRECTORY [<dir>]           |
                TARGET    [<target1> ...]   |
                SOURCE    [<src1> ...]
                          [DIRECTORY <dirs> ...]
                          [TARGET_DIRECTORY <targets> ...] |
                INSTALL   [<file1> ...]     |
                TEST      [<test1> ...]
                          [DIRECTORY <dir>] |
                CACHE     [<entry1> ...]    >
               [APPEND] [APPEND_STRING]
               PROPERTY <name> [<value1> ...])

スコープにある0個以上のオブジェクトに1個のプロパティをセットします。

最初の引数は、プロパティをセットするスコープを表します。
次のいずれかを指定して下さい：

``GLOBAL``
  「:ref:`グローバルのプロパティ <Global Properties>`」が対象。
  このスコープは一つだけしかなく、名前は付与できない。

``DIRECTORY``
  「:ref:`ディレクトリのプロパティ <Directory Properties>`」が対象。
  デフォルトは現在の作業ディレクトリ。
  CMake が認識している他のディレクトリ（絶対パスまたは相対パス）もスコープに指定できる。
  なお相対パスは :variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとしてパスを計算する。
  :command:`set_directory_properties` コマンドも参照のこと。

  .. versionadded:: 3.19
    ``<dir>`` に :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できるようになった。

``TARGET``
  「:ref:`ターゲットのプロパティ <Target Properties>`」が対象。
  0個以上のビルド・ターゲットをスコープに指定できる。
  :command:`set_target_properties` コマンドも参照のこと。

  ただし「:ref:`Alias Targets`」にプロパティはセットできない。

``SOURCE``
  「:ref:`ソース・ファイルのプロパティ <Source File Properties>`」が対象。
  0個以上の既存のソース・ファイルをスコープに指定できる。
  このコマンドを呼び出した ``CMakeLists.txt`` と同じディレクトリに追加したソース・ファイルが対象である。

  .. versionadded:: 3.18
    次のサブ・オプションの一つ、または両方を使って、他のディレクトリ・スコープで Visibility をセットできるようになった。

    ``DIRECTORY <dirs>...``
      ``<dirs>`` のディレクトリごとのスコープでソース・ファイルに対するプロパティをセットする。
      ``<dirs>`` には :command:`add_subdirectory` コマンドで追加したディレクトリか、:variable:`CMAKE_CURRENT_SOURCE_DIR` を指定する（つまり CMake が既に認識しているディレクトリ）。
      なお相対パスは :variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとしてパスを計算する。

      .. versionadded:: 3.19
        ``<dirs>`` に :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できるようになった。

    ``TARGET_DIRECTORY <targets>...``
      ソース・ファイルのプロパティを、 ``<targets> ...`` が作成したディレクトリのスコープごとにセットする（すなわち、既に ``<target>`` が存在している必要があり）。

  :command:`set_source_files_properties` コマンドも参照のこと。

``INSTALL``
  「:ref:`インストールしたファイルのプロパティ <Installed File Properties>`」が対象。

  .. versionadded:: 3.1

  0個以上のインストールしたファイル（のパス）をスコープに指定できる。
  これらのプロパティはインストール先に展開する際も影響があるので、:manual:`cpack <cpack(1)>` で利用できる。

  プロパティのキーと値は共に :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を使うことができる。
  特定のプロパティはインストール済みのファイルやディレクトリ（または両方）に適用される場合がある。

  ``<file>`` のパス名を構成する要素はスラッシュ（``/``）で区切り、:ref:`正規化 <Normalization>` しておくこと（さらに、大文字と小文字を区別するので注意のこと）。

  相対パスでインストール先の ``<prefix>`` を参照する場合はドット（``.``）を使うこと。

  現在インストールされているファイルのプロパティは、パス名がインストール先の ``<prefix>`` を基準とする WIX ジェネレータ（:cpack_gen:`CPack WIX Generator`）に対してのみ定義される（FIXME: 意味不明）。

``TEST``
  「:ref:`テストのプロパティ <Test Properties>`」が対象。
  プロパティのスコープは :manual:`ctest(1)` コマンドが呼び出されるディレクトリに制限される。
  0個以上のテストをスコープに指定できる。
  :command:`set_tests_properties` コマンドも参照のこと。

  このプロパティの値は  :command:`add_test(NAME)` コマンドで生成されたテストの :manual:`ジェネレータ式 <cmake-generator-expressions(7)>` を使って指定できる。

  .. versionadded:: 3.28

    次のサブ・オプションを使って、他のディレクトリ・スコープで Visibility をセットできるようになった。

    ``DIRECTORY <dir>``
      テストのプロパティを ``<dir>`` のディレクトリのスコープでセットする。
      ``<dir>`` には :command:`add_subdirectory` コマンドで追加したディレクトリか、:variable:`CMAKE_CURRENT_SOURCE_DIR` を指定する（つまり CMake が既に認識しているディレクトリ）。
      なお相対パスは :variable:`CMAKE_CURRENT_SOURCE_DIR` をベース・ディレクトリとしてパスを計算する。
      ``<dir>`` には :variable:`CMAKE_CURRENT_BINARY_DIR` を指定できる。

``CACHE``
  「:ref:`キャッシュ変数のプロパティ <Cache Entry Properties>`」が対象。
  0個以上の既存のキャッシュ・エントリをスコープに指定できる。

必須の ``PROPERTY`` オプションの直後は、セットするプロパティの名前と値が続き、それ以降は「:ref:`セミコロンで区切られたリスト <CMake Language Lists>` 」の書式で同様にプロパティの名前とその値のペアが続きます。

``APPEND`` オプションを指定すると、ここで指定した「:ref:`セミコロンで区切られたリスト <CMake Language Lists>` 」は既存のプロパティの定義の後ろに追加されます（ただし空の値は追加しません）。
``APPEND_STRING`` オプションを指定すると、指定したプロパティの値は文字列として追加されます（すなわち文字列のリストではなく、より長い文字列に連結して追加されます）。
``INHERITED`` をサポートするプロパティ（:command:`define_property` コマンド参照）に対して ``APPEND`` や ``APPEND_STRING`` を指定すると、追加するプロパティの値を見つける時に継承は行われません（FIXME: 意味不明）。
``APPEND`` や ``APPEND_STRING`` オプションを指定した時、対象のスコープで、まだプロパティが（継承ではなく）直接セットされていない場合、これらのオプションを無視します。

.. note::

  :prop_sf:`GENERATED` なソース・ファイルのプロパティはグローバルで表示される場合がある。
  詳細は、このドキュメントを参照のこと。

参考情報
^^^^^^^^

* :command:`define_property`
* :command:`get_property`
* :manual:`スコープごとのプロパティの一覧 <cmake-properties(7)>`
