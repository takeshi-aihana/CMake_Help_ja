CMake チュートリアル
********************

はじめに
========

CMake チュートリアルでは、一般的なビルド・システムの課題について CMake で解決するやり方をステップ毎に説明するガイドを提供しています。
サンプル・プロジェクトの中で、さまざまなトピックスがどのように連携しているのかを確認するのは、とても有用なことです。

ステップ
========

.. include:: source.txt

|tutorial_source|
Each step has its own subdirectory containing code that may be used as a starting point.
The tutorial examples are progressive so that each step provides the complete solution for the previous step.

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
