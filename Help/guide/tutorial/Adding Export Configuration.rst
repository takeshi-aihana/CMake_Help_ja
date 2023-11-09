ステップ11: プロジェクトの構成をエキスポートする
================================================

チュートリアルにあった「:guide:`tutorial/Installing and Testing`」では、CMake がプロジェクトのライブラリとヘッダ・ファイルをローカルにインストールする機能を追加しました。
また「:guide:`tutorial/Packaging an Installer`」のチュートリアルでは、これらのファイルをパッケージ化して他の人たちに配布できるようにする機能を追加しました。

この次のステップでは、他の CMake のプロジェクトで我々のプロジェクトをいろいろな場面 [#hint_for_using_our_project]_ で利用するために必要な情報を追加することにします。

最初のステップは :command:`install(TARGETS)` コマンドを変更して、``DESTINATION`` だけではなく ``EXPORT`` も指定するようにします。
``EXPORT`` というキーワードは、インストール・コマンドの対象となる全てのファイルをインストール先から取得するコードを CMake ファイルを出力します。
それでは、``MathFunctions/CMakeLists.txt`` の ``install`` コマンドを次のように変更して、``MathFunctions`` ライブラリを明示歴に ``EXPORT`` してみましょう：

.. literalinclude:: Complete/MathFunctions/CMakeLists.txt
  :caption: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-install-TARGETS-EXPORT
  :language: cmake
  :start-after: # install libs

ここで ``MathFunctions`` ライブラリをエキスポートして生成された ``MathFunctionsTargets.cmake`` という CMake ファイルを明示的にインストールする必要があります。
これはプロジェクト最上位にある ``CMakeLists.txt`` の最後に次のコマンドを追加することで実現できます：

.. literalinclude:: Complete/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-install-EXPORT
  :language: cmake
  :start-after: # install the configuration targets
  :end-before: include(CMakePackageConfigHelpers)

この時点で、CMake を実行して確認してみて下さい。
もし全ての設定が正しければ、CMake が次のようなエラーを出力するはずです：

.. code-block:: console

  Target "MathFunctions" INTERFACE_INCLUDE_DIRECTORIES property contains
  path:

    "/Users/robert/Documents/CMakeClass/Tutorial/Step11/MathFunctions"

  which is prefixed in the source directory.

このエラーで CMake が言おうとしていることは、エキスポートする情報にはそれを生成したマシンに固有の情報が含まれている、すなわち他のマシンには存在しないパス名もエキスポートしようとしたということです。
これを解決するには、``MathFunctions`` ライブラリの :command:`target_include_directories` コマンドを変更して、ビルド・ディレクトリの中からファイルを参照する場合と、インストール先（またはパッケージの中）からファイルを参照する場合とで、``INTERFACE`` が指す場所が違うということを CMake に理解させる必要があります。
これは、``MathFunctions`` ライブラリの :command:`target_include_directories` コマンド呼び出しを次のように変更するということです：

.. literalinclude:: Step12/MathFunctions/CMakeLists.txt
  :caption: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-target_include_directories
  :language: cmake
  :start-after: # to find MathFunctions.h, while we don't.
  :end-before: # should we use our own math functions

これを変更したら、もう一度 CMake を実行して、エラーが表示されなくなったことを確認して下さい。

この時点で、CMake はインストールするファイルの情報を正しくエキスポートするようになりますが、他のプロジェクトからそれらを :command:`find_package` コマンドで見つけられるようにするために、依然として ``MathFunctionsConfig.cmake`` の生成が必要です。
それではプロジェクト最上位に、次の内容を含む ``Config.cmake.in`` という新しいファイルを追加してみて下さい：

.. literalinclude:: Step12/Config.cmake.in
  :caption: Config.cmake.in
  :name: Config.cmake.in

それからプロジェクトを適切に構成し、ファイルをインストールすために、次のコマンドをプロジェクト最上位にある ``CMakeLists.txt`` の最後に追加して下さい：

.. literalinclude:: Step12/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-install-Config.cmake
  :language: cmake
  :start-after: # install the configuration targets
  :end-before: # generate the config file

そして :command:`configure_package_config_file` コマンドを呼び出します。
このコマンドは標準の :command:`configure_file` と同様に、指定されたファイルを設定しますがいくつか違いがあります。
このコマンドを適切に利用するには、そのファイルに必要な情報に加えて、``@PACKAGE_INIT@`` というキーワードだけを含んだ単一行が必要です。
このキーワードは、設定値を相対バスに変換するコードで置き換えられます。
ここで置き換えられた値は同じ名前で参照できますが、``PACKAGE_`` という接頭詞が付加されます。

.. literalinclude:: Step12/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-configure-package-config.cmake
  :language: cmake
  :start-after: # install the configuration targets
  :end-before: # generate the version file

次は :command:`write_basic_package_version_file` コマンドです。
このコマンドは :command:`find_package` コマンドで使用されるファイルを作成し、必要なパッケージのバージョンと互換性を記録します。
ここでは ``Tutorial_VERSION_*`` という変数を使って ``AnyNewerVersion`` と互換性があることを宣言します。
これは、このバージョンまたはこれ以降のバージョンは、要求されたバージョンと互換性があることを意味します。

.. literalinclude:: Step12/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-basic-version-file.cmake
  :language: cmake
  :start-after: # generate the version file
  :end-before: # install the generated configuration files

最後に、生成された両方のファイルがインストールされるように設定します：

.. literalinclude:: Step12/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-install-configured-files.cmake
  :language: cmake
  :start-after: # install the generated configuration files
  :end-before: # generate the export

この時点で、他のプロジェクトをインストールまたはパッケージ化したあとで利用することが可能な、我々のプロジェクト向けの（再配置が可能な）CMake の構成情報が生成されるようになりました。
他のプロジェクトで、我々のプロジェクトをビルド・ディレクトリから利用したい場合は、そのプロジェクト最上位にある ``CMakeLists.txt`` の最後に次のコマンドを追加するだけです：

.. literalinclude:: Step12/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-export
  :language: cmake
  :start-after: # needs to be after the install(TARGETS) command


:command:`export` コマンド呼び出しで ``MathFunctionsTargets.cmake`` が生成され、ビルド・ディレクトリにある構成済みの ``MathFunctionsConfig.cmake`` ファイルをインストールすることなしに、他のプロジェクトで利用できるようになります。

.. rubric:: 日本語訳注記

.. [#hint_for_using_our_project] たとえばビルド時とか、ローカルにインストールする時とか、あるいはパッケージされた時に利用するなど。
