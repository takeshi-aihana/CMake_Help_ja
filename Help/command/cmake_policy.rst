cmake_policy
------------

CMake プロジェクトのポリシーを管理する。
これまで定義されたポリシーとその設定については :manual:`cmake-policies(7)` のマニュアルを参照のこと。

CMake が進化するにつれて、バグを修正したり、既存の機能を改善するための変更が必要になることがある。
「CMake のポリシー・メカニズム」は、新しいバージョンの CMake で動作が変更になった時に、古いバージョンで生成したプロジェクトのビルドを継続できるよう設計されている。
動作の変更、すなわち「**ポリシー**」が新しくなると ``CMP<NNNN>`` という識別子が付与される（ ``<NNNN>`` は整数のインデックス）。
それぞれのポリシーに関連付けられたドキュメントには ``OLD`` と ``NEW`` のカテゴリと、ポリシーが導入された理由が説明されている。
CMake のプロジェクトは希望する機能を選択するためにポリシーを設定できる。
CMake は、プロジェクトでどの機能を使用するのかを知る必要がある時、そのポリシーの設定を確認する。
ポリシーが設定されていない場合は ``OLD`` のカテゴリに属す動作が期待されていると想定し、ポリシーを設定するように警告が発せられる。

CMake のバージョンごとのポリシー
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The ``cmake_policy`` command is used to set policies to ``OLD`` or ``NEW`` behavior.
While setting policies individually is supported, we encourage projects to set policies based on CMake versions:

.. signature:: cmake_policy(VERSION <min>[...<max>])
  :target: VERSION

.. versionadded:: 3.12
  The optional ``<max>`` version.

``<min>`` and the optional ``<max>`` are each CMake versions of the form ``major.minor[.patch[.tweak]]``, and the ``...`` is literal.
The ``<min>`` version must be at least ``2.4`` and at most the running version of CMake.
The ``<max>`` version, if specified, must be at least the ``<min>`` version but may exceed the running version of CMake.
If the running version of CMake is older than 3.12, the extra ``...`` dots will be seen as version component separators, resulting in the ``...<max>`` part being ignored and preserving the pre-3.12 behavior of basing policies on ``<min>``.

This specifies that the current CMake code is written for the given range of CMake versions.
All policies known to the running version of CMake and introduced in the ``<min>`` (or ``<max>``, if specified) version or earlier will be set to use ``NEW`` behavior.
All policies introduced in later versions will be unset (unless the :variable:`CMAKE_POLICY_DEFAULT_CMP<NNNN>` variable sets a default).
This effectively requests behavior preferred as of a given CMake version and tells newer CMake versions to warn about their new policies.

Note that the :command:`cmake_minimum_required(VERSION)` command implicitly calls ``cmake_policy(VERSION)`` too.

.. include:: DEPRECATED_POLICY_VERSIONS.txt

Setting Policies Explicitly
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. signature:: cmake_policy(SET CMP<NNNN> NEW|OLD)
  :target: SET

Tell CMake to use the ``OLD`` or ``NEW`` behavior for a given policy.
Projects depending on the old behavior of a given policy may silence a
policy warning by setting the policy state to ``OLD``.  Alternatively
one may fix the project to work with the new behavior and set the
policy state to ``NEW``.

.. include:: ../policy/DEPRECATED.txt

Checking Policy Settings
^^^^^^^^^^^^^^^^^^^^^^^^

.. signature:: cmake_policy(GET CMP<NNNN> <variable>)
  :target: GET

Check whether a given policy is set to ``OLD`` or ``NEW`` behavior.
The output ``<variable>`` value will be ``OLD`` or ``NEW`` if the
policy is set, and empty otherwise.

CMake Policy Stack
^^^^^^^^^^^^^^^^^^

CMake keeps policy settings on a stack, so changes made by the
``cmake_policy`` command affect only the top of the stack.  A new entry on
the policy stack is managed automatically for each subdirectory to
protect its parents and siblings.  CMake also manages a new entry for
scripts loaded by :command:`include` and :command:`find_package` commands
except when invoked with the ``NO_POLICY_SCOPE`` option
(see also policy :policy:`CMP0011`).
The ``cmake_policy`` command provides an interface to manage custom
entries on the policy stack:

.. signature:: cmake_policy(PUSH)
  :target: PUSH

  Create a new entry on the policy stack.

.. signature:: cmake_policy(POP)
  :target: POP

  Remove the last policy stack entry created with ``cmake_policy(PUSH)``.

Each ``PUSH`` must have a matching ``POP`` to erase any changes.
This is useful to make temporary changes to policy settings.
Calls to the :command:`cmake_minimum_required(VERSION)`,
:command:`cmake_policy(VERSION)`, or :command:`cmake_policy(SET)` commands
influence only the current top of the policy stack.

.. versionadded:: 3.25
  The :command:`block(SCOPE_FOR POLICIES)` command offers a more flexible
  and more secure way to manage the policy stack. The pop action is done
  automatically when leaving the block scope, so there is no need to
  precede each :command:`return` with a call to :command:`cmake_policy(POP)`.

  .. code-block:: cmake

    # stack management with cmake_policy()
    function(my_func)
      cmake_policy(PUSH)
      cmake_policy(SET ...)
      if (<cond1>)
        ...
        cmake_policy(POP)
        return()
      elseif(<cond2>)
        ...
        cmake_policy(POP)
        return()
      endif()
      ...
      cmake_policy(POP)
    endfunction()

    # stack management with block()/endblock()
    function(my_func)
      block(SCOPE_FOR POLICIES)
        cmake_policy(SET ...)
        if (<cond1>)
          ...
          return()
        elseif(<cond2>)
          ...
          return()
        endif()
        ...
      endblock()
    endfunction()

Commands created by the :command:`function` and :command:`macro`
commands record policy settings when they are created and
use the pre-record policies when they are invoked.  If the function or
macro implementation sets policies, the changes automatically
propagate up through callers until they reach the closest nested
policy stack entry.

参考情報
^^^^^^^^

* :command:`cmake_minimum_required`
