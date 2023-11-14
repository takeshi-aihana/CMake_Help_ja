CMake チュートリアル
********************

はじめに
========

CMake チュートリアルでは、一般的なビルドシステムの課題について CMake で解決する方法を順を追って説明するガイドを提供しています。
サンプル・プロジェクトの中で、さまざまなトピックスがどのように連携しているのかを確認することは、とても役に立ちます。

ステップ
========

.. include:: source.txt

|tutorial_source|
実践するステップごとに専用のサブディレクトリがあり、その中には出発点として利用できるサンプル・コードがあります。
チュートリアルにあるサンプルをステップ・バイ・ステップで説明しているので、各ステップはその前のステップの解決方法が含まれます。

.. toctree::
  :maxdepth: 2

  A Basic Starting Point
  Adding a Library
  Adding Usage Requirements for a Library
  Adding Generator Expressions
  Installing and Testing
  Adding Support for a Testing Dashboard
  Adding System Introspection
  Adding a Custom Command and Generated File
  Packaging an Installer
  Selecting Static or Shared Libraries
  Adding Export Configuration
  Packaging Debug and Release

..
  Whenever a step above is renamed or removed, leave forwarding text in
  its original document file, and list it below to preserve old links
  to cmake.org/cmake/help/latest/ URLs.

.. toctree::
  :maxdepth: 1
  :hidden:
