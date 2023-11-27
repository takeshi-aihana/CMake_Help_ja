cmake_language
--------------

.. versionadded:: 3.18

CMake コマンドでメタ操作（*meta-operations*）を呼び出す。

概要
^^^^

.. parsed-literal::

  cmake_language(`CALL`_ <command> [<arg>...])
  cmake_language(`EVAL`_ CODE <code>...)
  cmake_language(`DEFER`_ <options>... CALL <command> [<arg>...])
  cmake_language(`SET_DEPENDENCY_PROVIDER`_ <command> SUPPORTED_METHODS <methods>...)
  cmake_language(`GET_MESSAGE_LOG_LEVEL`_ <out-var>)

はじめに
^^^^^^^^

このコマンドは内蔵の CMake コマンド、または :command:`macro` や :command:`function` コマンドで作成したメタ操作を呼び出します。

``cmake_language`` は新しい変数やポリシーのスコープを導入しません。

コマンドの呼び出し
^^^^^^^^^^^^^^^^^^

.. signature::
  cmake_language(CALL <command> [<arg>...])

  引数を指定した場合、それを使って名前付きの ``<command>`` を呼び出す。
  たとえば、このメタ操作は：

  .. code-block:: cmake

    set(message_command "message")
    cmake_language(CALL ${message_command} STATUS "Hello World!")

  次と等価である：

  .. code-block:: cmake

    message(STATUS "Hello World!")

  .. note::
    コードの一貫性を保証するため、次のような操作は指定しないこと：

    * ``if`` / ``elseif`` / ``else`` / ``endif``
    * ``block`` / ``endblock``
    * ``while`` / ``endwhile``
    * ``foreach`` / ``endforeach``
    * ``function`` / ``endfunction``
    * ``macro`` / ``endmacro``

コードの評価
^^^^^^^^^^^^

.. signature::
  cmake_language(EVAL CODE <code>...)
  :target: EVAL

  ``<code>...`` を CMake のコードとして評価する。

  たとえば、このコードは：

  .. code-block:: cmake

    set(A TRUE)
    set(B TRUE)
    set(C TRUE)
    set(condition "(A AND B) OR C")

    cmake_language(EVAL CODE "
      if (${condition})
        message(STATUS TRUE)
      else()
        message(STATUS FALSE)
      endif()"
    )

  次と等価である：

  .. code-block:: cmake

    set(A TRUE)
    set(B TRUE)
    set(C TRUE)
    set(condition "(A AND B) OR C")

    file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/eval.cmake "
      if (${condition})
        message(STATUS TRUE)
      else()
        message(STATUS FALSE)
      endif()"
    )

    include(${CMAKE_CURRENT_BINARY_DIR}/eval.cmake)

遅延呼び出し
^^^^^^^^^^^^

.. versionadded:: 3.19

.. signature::
  cmake_language(DEFER <options>... CALL <command> [<arg>...])

  引数を指定していれば、それを使った名前付きの ``<command>`` を、あとで呼び出すようスケジュールする。  
  デフォルトで、このような遅延呼び出しは、それらが :command:`return()` コマンドの呼び出しのあとでも実行されることを除き、現在のディレクトリにある CMakeLists.txt ファイルの最後に書かれたコマンドであるかのように実行される。
  なお、引数として渡した変数の参照は、遅延呼び出しとして実際に実行する時に評価される。

  指定できるオプションは次の通り：

  ``DIRECTORY <dir>``
    現在のソース・ディレクトリではなく ``<dir>`` の最後で ``<command>`` が呼び出されるようにスケジュールする。
    引数の ``<dir>`` はソース・ディレクトリ、またはそれに対応するバイナリ・ディレクトリを参照できる。
    相対パスは、現在のソース・ディレクトリからの相対ディレクトリとして扱われる。

    CMake は ``<dir>`` をプロジェクト最上位のディレクトリか、または :command:`add_subdirectory` コマンドで追加したディレクトリのいずれかとして認識しておく必要がある。
    さらに、``<dir>`` に対する処理が完了していないこと。
    これは、``<dir>`` が現在のソース・ディレクトリまたはその上のディレクトリであることを意味する。

  ``ID <id>``
    遅延呼び出しを識別する ID を指定する。
    ``<id>`` は空にしないこと。さらに先頭文字を大文字の ``A-Z`` にしないこと。
    ただし、このオプションを使用せずに ``ID_VAR`` というオプションを利用して自動的に ``<id>`` が生成された場合にだけ、先頭文字をアンダースコア（``_``）にすることができる。

  ``ID_VAR <var>``
    遅延呼び出しの ID を格納する変数を指定する。
    もしオプション ``ID <id>`` が指定されていない場合は、先頭文字をアンダースコア（``_``）とする新しい ID を生成する。

  現在、遅延されている呼び出しのリストは次のようにして取得できます：

  .. code-block:: cmake

    cmake_language(DEFER [DIRECTORY <dir>] GET_CALL_IDS <var>)

  この呼び出しにより、:ref:`セミコロンで区切られた遅延呼び出しの ID を要素とするリスト <CMake Language Lists>` が ``<var>`` に格納されます。
  これらの ID は呼び出しが遅延されたディレクトリのスコープ（つまり、遅延された呼び出しが実行される場所）に対して付与された識別子で、その呼び出しが生成されたスコープとは異なる場合があります。
  ``DIRECTORY`` オプションを利用して、取得する ID のスコープを指定できます。
  この ``DIRECTORY`` オプションが指定されていない場合、現在のディレクトリをスコープとする遅延呼び出しのID を返します。

  特定の遅延呼び出しの詳細は ID から取得できます：

  .. code-block:: cmake

    cmake_language(DEFER [DIRECTORY <dir>] GET_CALL <id> <var>)

  この呼び出しにより、:ref:`セミコロンで区切られた遅延呼び出しの ID を要素とするリスト <CMake Language Lists>` が ``<var>`` に格納され、その先頭の要素が呼び出すコマンドの名前で、残りの要素は未評価の引数です
  （注意：この引数に含まれている ``;`` という文字は文字通りに評価されるため、シェルのような複数の引数を表す文字とは区別できません）。
  もし複数回の呼び出しが同じ ID でスケジューリングされている場合は、１番目の呼び出しを取得します。
  もし指定した ``DIRECTORY`` スコープで、与えられた ID に対して何も呼び出しがスケジューリングされていない場合（あるいは、``DIRECTORY`` オプションが指定されていないため、現在のディレクトリがスコープになっている場合で）、空の文字列が ``<var>`` に格納されます。

  遅延呼び出しはその ID を使ってキャンセルできます：

  .. code-block:: cmake

    cmake_language(DEFER [DIRECTORY <dir>] CANCEL_CALL <id>...)

  この呼び出しにより、指定した ``DIRECTORY`` のスコープ（あるいは、``DIRECTORY`` オプションが指定されていないため現在のディレクトリがスコープになっている場合）でいずれかの ``<id>`` に一致する全ての遅延呼び出しがキャンセルされます。
  マッチしない ``<id>`` は何も表示せずに無視されます。

遅延呼び出しの例
""""""""""""""""

例えば、次のコード：

.. code-block:: cmake

  cmake_language(DEFER CALL message "${deferred_message}")
  cmake_language(DEFER ID_VAR id CALL message "Canceled Message")
  cmake_language(DEFER CANCEL_CALL ${id})
  message("Immediate Message")
  set(deferred_message "Deferred Message")

の実行結果は::

  Immediate Message
  Deferred Message

``Cancelled Message`` という文字列は、それを遅延呼び出しするコマンドがキャンセルされているので表示されません。
``deferred_message`` という変数はその値がセットされるまで評価されることはありません。そのため遅延呼び出しを定義したあとにセットできます。

遅延呼び出しをスケジューリングした時に変数をすぐに評価するには、その呼び出しのコードを ``cmake_language(EVAL)`` でラップして下さい。
ここで、このコマンドの引数は遅延呼び出しされた時に再評価されて上書きされてしまうことに注意して下さい。その場合、その引数をカッコ（``[[...]]``）でくくることで回避できます。
たとえば:

.. code-block:: cmake

  set(deferred_message "Deferred Message 1")
  set(re_evaluated [[${deferred_message}]])
  cmake_language(EVAL CODE "
    cmake_language(DEFER CALL message [[${deferred_message}]])
    cmake_language(DEFER CALL message \"${re_evaluated}\")
  ")
  message("Immediate Message")
  set(deferred_message "Deferred Message 2")

の実行結果は::

  Immediate Message
  Deferred Message 1
  Deferred Message 2

.. _dependency_providers:

Dependency Providers
^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.24

.. note:: A high-level introduction to this feature can be found in the
          :ref:`Using Dependencies Guide <dependency_providers_overview>`.

.. signature::
  cmake_language(SET_DEPENDENCY_PROVIDER <command>
                 SUPPORTED_METHODS <methods>...)

  When a call is made to :command:`find_package` or
  :command:`FetchContent_MakeAvailable`, the call may be forwarded to a
  dependency provider which then has the opportunity to fulfill the request.
  If the request is for one of the ``<methods>`` specified when the provider
  was set, CMake calls the provider's ``<command>`` with a set of
  method-specific arguments.  If the provider does not fulfill the request,
  or if the provider doesn't support the request's method, or no provider
  is set, the built-in :command:`find_package` or
  :command:`FetchContent_MakeAvailable` implementation is used to fulfill
  the request in the usual way.

  One or more of the following values can be specified for the ``<methods>``
  when setting the provider:

  ``FIND_PACKAGE``
    The provider command accepts :command:`find_package` requests.

  ``FETCHCONTENT_MAKEAVAILABLE_SERIAL``
    The provider command accepts :command:`FetchContent_MakeAvailable`
    requests.  It expects each dependency to be fed to the provider command
    one at a time, not the whole list in one go.

  Only one provider can be set at any point in time.  If a provider is already
  set when ``cmake_language(SET_DEPENDENCY_PROVIDER)`` is called, the new
  provider replaces the previously set one.  The specified ``<command>`` must
  already exist when ``cmake_language(SET_DEPENDENCY_PROVIDER)`` is called.
  As a special case, providing an empty string for the ``<command>`` and no
  ``<methods>`` will discard any previously set provider.

  The dependency provider can only be set while processing one of the files
  specified by the :variable:`CMAKE_PROJECT_TOP_LEVEL_INCLUDES` variable.
  Thus, dependency providers can only be set as part of the first call to
  :command:`project`.  Calling ``cmake_language(SET_DEPENDENCY_PROVIDER)``
  outside of that context will result in an error.

  .. note::
    The choice of dependency provider should always be under the user's control.
    As a convenience, a project may choose to provide a file that users can
    list in their :variable:`CMAKE_PROJECT_TOP_LEVEL_INCLUDES` variable, but
    the use of such a file should always be the user's choice.

Provider commands
"""""""""""""""""

Providers define a single ``<command>`` to accept requests.  The name of
the command should be specific to that provider, not something overly
generic that another provider might also use.  This enables users to compose
different providers in their own custom provider.  The recommended form is
``xxx_provide_dependency()``, where ``xxx`` is the provider-specific part
(e.g. ``vcpkg_provide_dependency()``, ``conan_provide_dependency()``,
``ourcompany_provide_dependency()``, and so on).

.. code-block:: cmake

  xxx_provide_dependency(<method> [<method-specific-args>...])

Because some methods expect certain variables to be set in the calling scope,
the provider command should typically be implemented as a macro rather than a
function.  This ensures it does not introduce a new variable scope.

The arguments CMake passes to the dependency provider depend on the type of
request.  The first argument is always the method, and it will only ever
be one of the ``<methods>`` that was specified when setting the provider.

``FIND_PACKAGE``
  The ``<method-specific-args>`` will be everything passed to the
  :command:`find_package` call that requested the dependency.  The first of
  these ``<method-specific-args>`` will therefore always be the name of the
  dependency.  Dependency names are case-sensitive for this method because
  :command:`find_package` treats them case-sensitively too.

  If the provider command fulfills the request, it must set the same variable
  that :command:`find_package` expects to be set.  For a dependency named
  ``depName``, the provider must set ``depName_FOUND`` to true if it fulfilled
  the request.  If the provider returns without setting this variable, CMake
  will assume the request was not fulfilled and will fall back to the
  built-in implementation.

  If the provider needs to call the built-in :command:`find_package`
  implementation as part of its processing, it can do so by including the
  ``BYPASS_PROVIDER`` keyword as one of the arguments.

``FETCHCONTENT_MAKEAVAILABE_SERIAL``
  The ``<method-specific-args>`` will be everything passed to the
  :command:`FetchContent_Declare` call that corresponds to the requested
  dependency, with the following exceptions:

  * If ``SOURCE_DIR`` or ``BINARY_DIR`` were not part of the original
    declared arguments, they will be added with their default values.
  * If :variable:`FETCHCONTENT_TRY_FIND_PACKAGE_MODE` is set to ``NEVER``,
    any ``FIND_PACKAGE_ARGS`` will be omitted.
  * The ``OVERRIDE_FIND_PACKAGE`` keyword is always omitted.

  The first of the ``<method-specific-args>`` will always be the name of the
  dependency.  Dependency names are case-insensitive for this method because
  :module:`FetchContent` also treats them case-insensitively.

  If the provider fulfills the request, it should call
  :command:`FetchContent_SetPopulated`, passing the name of the dependency as
  the first argument.  The ``SOURCE_DIR`` and ``BINARY_DIR`` arguments to that
  command should only be given if the provider makes the dependency's source
  and build directories available in exactly the same way as the built-in
  :command:`FetchContent_MakeAvailable` command.

  If the provider returns without calling :command:`FetchContent_SetPopulated`
  for the named dependency, CMake will assume the request was not fulfilled
  and will fall back to the built-in implementation.

  Note that empty arguments may be significant for this method (e.g. an empty
  string following a ``GIT_SUBMODULES`` keyword).  Therefore, if forwarding
  these arguments on to another command, extra care must be taken to avoid such
  arguments being silently dropped.

  If ``FETCHCONTENT_SOURCE_DIR_<uppercaseDepName>`` is set, then the
  dependency provider will never see requests for the ``<depName>`` dependency
  for this method. When the user sets such a variable, they are explicitly
  overriding where to get that dependency from and are taking on the
  responsibility that their overriding version meets any requirements for that
  dependency and is compatible with whatever else in the project uses it.
  Depending on the value of :variable:`FETCHCONTENT_TRY_FIND_PACKAGE_MODE`
  and whether the ``OVERRIDE_FIND_PACKAGE`` option was given to
  :command:`FetchContent_Declare`, having
  ``FETCHCONTENT_SOURCE_DIR_<uppercaseDepName>`` set may also prevent the
  dependency provider from seeing requests for a ``find_package(depName)``
  call too.

Provider Examples
"""""""""""""""""

This first example only intercepts :command:`find_package` calls.  The
provider command runs an external tool which copies the relevant artifacts
into a provider-specific directory, if that tool knows about the dependency.
It then relies on the built-in implementation to then find those artifacts.
:command:`FetchContent_MakeAvailable` calls would not go through the provider.

.. code-block:: cmake
  :caption: mycomp_provider.cmake

  # Always ensure we have the policy settings this provider expects
  cmake_minimum_required(VERSION 3.24)

  set(MYCOMP_PROVIDER_INSTALL_DIR ${CMAKE_BINARY_DIR}/mycomp_packages
    CACHE PATH "The directory this provider installs packages to"
  )
  # Tell the built-in implementation to look in our area first, unless
  # the find_package() call uses NO_..._PATH options to exclude it
  list(APPEND CMAKE_MODULE_PATH ${MYCOMP_PROVIDER_INSTALL_DIR}/cmake)
  list(APPEND CMAKE_PREFIX_PATH ${MYCOMP_PROVIDER_INSTALL_DIR})

  macro(mycomp_provide_dependency method package_name)
    execute_process(
      COMMAND some_tool ${package_name} --installdir ${MYCOMP_PROVIDER_INSTALL_DIR}
      COMMAND_ERROR_IS_FATAL ANY
    )
  endmacro()

  cmake_language(
    SET_DEPENDENCY_PROVIDER mycomp_provide_dependency
    SUPPORTED_METHODS FIND_PACKAGE
  )

The user would then typically use the above file like so::

  cmake -DCMAKE_PROJECT_TOP_LEVEL_INCLUDES=/path/to/mycomp_provider.cmake ...

The next example demonstrates a provider that accepts both methods, but
only handles one specific dependency.  It enforces providing Google Test
using :module:`FetchContent`, but leaves all other dependencies to be
fulfilled by CMake's built-in implementation.  It accepts a few different
names, which demonstrates one way of working around projects that hard-code
an unusual or undesirable way of adding this particular dependency to the
build.  The example also demonstrates how to use the :command:`list` command
to preserve variables that may be overwritten by a call to
:command:`FetchContent_MakeAvailable`.

.. code-block:: cmake
  :caption: mycomp_provider.cmake

  cmake_minimum_required(VERSION 3.24)

  # Because we declare this very early, it will take precedence over any
  # details the project might declare later for the same thing
  include(FetchContent)
  FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        e2239ee6043f73722e7aa812a459f54a28552929 # release-1.11.0
  )

  # Both FIND_PACKAGE and FETCHCONTENT_MAKEAVAILABLE_SERIAL methods provide
  # the package or dependency name as the first method-specific argument.
  macro(mycomp_provide_dependency method dep_name)
    if("${dep_name}" MATCHES "^(gtest|googletest)$")
      # Save our current command arguments in case we are called recursively
      list(APPEND mycomp_provider_args ${method} ${dep_name})

      # This will forward to the built-in FetchContent implementation,
      # which detects a recursive call for the same thing and avoids calling
      # the provider again if dep_name is the same as the current call.
      FetchContent_MakeAvailable(googletest)

      # Restore our command arguments
      list(POP_BACK mycomp_provider_args dep_name method)

      # Tell the caller we fulfilled the request
      if("${method}" STREQUAL "FIND_PACKAGE")
        # We need to set this if we got here from a find_package() call
        # since we used a different method to fulfill the request.
        # This example assumes projects only use the gtest targets,
        # not any of the variables the FindGTest module may define.
        set(${dep_name}_FOUND TRUE)
      elseif(NOT "${dep_name}" STREQUAL "googletest")
        # We used the same method, but were given a different name to the
        # one we populated with. Tell the caller about the name it used.
        FetchContent_SetPopulated(${dep_name}
          SOURCE_DIR "${googletest_SOURCE_DIR}"
          BINARY_DIR "${googletest_BINARY_DIR}"
        )
      endif()
    endif()
  endmacro()

  cmake_language(
    SET_DEPENDENCY_PROVIDER mycomp_provide_dependency
    SUPPORTED_METHODS
      FIND_PACKAGE
      FETCHCONTENT_MAKEAVAILABLE_SERIAL
  )

The final example demonstrates how to modify arguments to a
:command:`find_package` call.  It forces all such calls to have the
``QUIET`` keyword.  It uses the ``BYPASS_PROVIDER`` keyword to prevent
calling the provider command recursively for the same dependency.

.. code-block:: cmake
  :caption: mycomp_provider.cmake

  cmake_minimum_required(VERSION 3.24)

  macro(mycomp_provide_dependency method)
    find_package(${ARGN} BYPASS_PROVIDER QUIET)
  endmacro()

  cmake_language(
    SET_DEPENDENCY_PROVIDER mycomp_provide_dependency
    SUPPORTED_METHODS FIND_PACKAGE
  )

message コマンドの現在のログ・レベルを取得する
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.25

.. _query_message_log_level:

.. signature::
  cmake_language(GET_MESSAGE_LOG_LEVEL <output_variable>)

  現在の :command:`message` コマンドによるログ・レベルを、指定した ``<output_variable>`` という変数に格納します。

  指定できるログのレベルについては :command:`message` コマンドを参照して下さい。

  ``message()`` コマンドの現在のログ・レベルを設定するには :manual:`cmake(1)` のコマンドライン・オプションである :option:`--log-level <cmake --log-level>` を使用するか、CMake 変数の :variable:`CMAKE_MESSAGE_LOG_LEVEL` を使います。

  コマンドライン・オプションと変数の両方が設定されれている場合は、コマンドライン・オプションで指定されたログ・レベルが格納されます。
  対して、どちらも設定されていない場合は、デフォルトのログ・レベルが格納されます。
