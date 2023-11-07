ステップ１０: 静的ライブラリや共有ライブラリを選択する
======================================================

このチュートリアルでは、:variable:`BUILD_SHARED_LIBS` という変数を利用して :command:`add_library` コマンドのデフォルトの挙動を制御する方法を確認して、明示的に種類（``STATIC``、``SHARED``、``MODULE`` 、または ``OBJECT``）が指定されていないライブラリのビルド方法を制御できるようにします。

これを実現するには、このプロジェクト最上位の ``CMakeLists.txt`` に変数 :variable:`BUILD_SHARED_LIBS` を追加する必要があります。
:command:`option` コマンドを使用して、ユーザがオプションとして値を ``ON`` / ``OFF`` できるようにします。

.. literalinclude:: Step11/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-option-BUILD_SHARED_LIBS
  :language: cmake
  :start-after: set(CMAKE_RUNTIME_OUTPUT_DIRECTORY
  :end-before: # configure a header file to pass the version number only

次に、静的ライブラリと共有ライブラリを出力するディレクトリを指定します。

.. literalinclude:: Step11/CMakeLists.txt
  :caption: CMakeLists.txt
  :name: CMakeLists.txt-cmake-output-directories
  :language: cmake
  :start-after: # we don't need to tinker with the path to run the executable
  :end-before: # configure a header file to pass the version number only

最後に ``MathFunctions/MathFunctions.h`` を修正して DLL の EXPORT 定義 [#hint_for_dllexport]_ を使用するようにします：

.. literalinclude:: Step11/MathFunctions/MathFunctions.h
  :caption: MathFunctions/MathFunctions.h
  :name: MathFunctions/MathFunctions.h
  :language: c++

この時点で、すべてをビルドすると PIC（*Position Independent Code* ：位置独立コード）を持たない静的ライブラリと PIC を持つ共有ライブラリをリンクしてエラーになることに気づくことでしょう。
これを解決するために、共有ライブラリの ``SqrtLibrary`` をビルドする際に :prop_tgt:`POSITION_INDEPENDENT_CODE` というターゲット・プロパティを明示的に ``True`` にしておきます。

.. literalinclude:: Step11/MathFunctions/CMakeLists.txt
  :caption: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-POSITION_INDEPENDENT_CODE
  :language: cmake
  :start-at: # state that SqrtLibrary need PIC when the default is shared libraries
  :end-at:  )

Windows でビルドするときは ``declspec(dllexport)``  を使用することを示す ``EXPORTING_MYMATH`` を定義して下さい [#hint_for_dllexport]_ 。

.. literalinclude:: Step11/MathFunctions/CMakeLists.txt
  :caption: MathFunctions/CMakeLists.txt
  :name: MathFunctions/CMakeLists.txt-dll-export
  :language: cmake
  :start-at: # define the symbol stating we are using the declspec(dllexport) when
  :end-at: target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")

**演習**:
上で、DLL の EXPORT 定義を利用するために ``MathFunctions.h`` ファイルを修正しました。
CMake のドキュメントを参照して、これを簡単に実現するヘルパー・モジュールを見つけてみて下さい。

.. rubric:: 日本語訳注記

.. [#hint_for_dllexport] `__declspec(dllexport) を使った DLL からのエクスポート <https://learn.microsoft.com/ja-jp/cpp/build/exporting-from-a-dll-using-declspec-dllexport?view=msvc-170>`_ 参照。
