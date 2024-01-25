message
-------

メッセージをログする。

概要
^^^^

.. parsed-literal::

  `一般的なメッセージ`_
    message([<mode>] "message text" ...)

  `チェックした結果を報告するメッセージ`_
    message(<checkState> "message text" ...)

  `Configure ログのメッセージ`_
    message(CONFIGURE_LOG <text>...)

一般的なメッセージ
^^^^^^^^^^^^^^^^^^

.. code-block:: cmake

  message([<mode>] "message text" ...)

ログに ``"message text"`` を記録します。
複数の ``"message text" ...`` を渡すと、区切り文字を挿入せずに一個のメッセージに連結して記録します。

``<mode>`` オプションでログするメッセージのレベル（種類）を指定します。このオプションはメッセージの処理方法に影響します：

``FATAL_ERROR``
  CMake のエラー。
  ビルド処理やファイルの生成を停止する。

  :manual:`cmake(1)` コマンドは 0 以外の「:ref:`終了コード <CMake Exit Code>`」を返す。

``SEND_ERROR``
  CMake のエラー。
  ビルド処理は続行するが、ファイルの生成はスキップする。

``WARNING``
  CMake のワーニング。
  ビルド処理は続行する。

``AUTHOR_WARNING``
  CMake のワーニング（dev）。
  ビルド処理は続行する。

``DEPRECATION``
  CMake 変数の :variable:`CMAKE_ERROR_DEPRECATED` または :variable:`CMAKE_WARN_DEPRECATED` が有効な場合は、CMake の非推奨エラーまたはワーニングをログし、それ以外のメッセージはログしない。

(none) または ``NOTICE``
  ユーザの注意を引くために、重要なメッセージを標準エラー出力にログする。

``STATUS``
  ユーザが関心を持つであろう情報をログする。
  理想的には、1行分の簡潔なメッセージにすること。

``VERBOSE``
  ユーザを対象とした詳細な情報をログする。
  これらのメッセージは、ほとんどの場合は直接ビルドとは関係のない追加の情報を提供するためのもので、開発者にとっては何が起こっているのか深い洞察が必要なケースに役立つ詳細情報になる場合がある。

``DEBUG``
  プロジェクトをビルドするだけのユーザではなく、プロジェクトに取り組んでいる開発者を対象とした詳細な情報をログする。
  こえらのメッセージは、プロジェクトをビルドしているユーザには興味あるものではなく、多くの場合は内部にある実装の詳細に密接に関係する情報である。

``TRACE``
  非常に低レベルの実装詳細を含んだきめの細かい情報をログする。
  このレベルのメッセージは、通常は一時的な情報であり、プロジェクトのリリースやパッケージ化などを行う前に削除されることが想定されている。

.. versionadded:: 3.15
  ``NOTICE`` と ``VERBOSE`` と ``DEBUG`` と ``TRACE`` レベルが追加された。

:manual:`cmake(1)` コマンドラインは ``STATUS`` から ``TRACE`` レベルのメッセージを、``"message text"`` の前に2個のハイフンとスペース（"``--`` "）を付けて、標準出力にログします。
それ以外のレベルのメッセージは全て標準エラー出力にログされ、先頭にハイフン（"``--``"）は付与されません。
All other message types are sent to stderr and are not prefixed with hyphens.
The :manual:`CMake GUI <cmake-gui(1)>` displays all messages in its log area.
The :manual:`curses interface <ccmake(1)>` shows ``STATUS`` to ``TRACE`` messages one at a time on a status line and other messages in an interactive pop-up box.
The :option:`--log-level <cmake --log-level>` command-line option to each of these tools can be used to control which messages will be shown.

.. versionadded:: 3.17
  To make a log level persist between CMake runs, the :variable:`CMAKE_MESSAGE_LOG_LEVEL` variable can be set instead.
  Note that the command line option takes precedence over the cache variable.

.. versionadded:: 3.16
  Messages of log levels ``NOTICE`` and below will have each line preceded by the content of the :variable:`CMAKE_MESSAGE_INDENT` variable (converted to a single string by concatenating its list items).
  For ``STATUS`` to ``TRACE`` messages, this indenting content will be inserted after the hyphens.

.. versionadded:: 3.17
  Messages of log levels ``NOTICE`` and below can also have each line preceded with context of the form ``[some.context.example]``.
  The content between the square brackets is obtained by converting the :variable:`CMAKE_MESSAGE_CONTEXT` list variable to a dot-separated string.
  The message context will always appear before any indenting content but after any automatically added leading hyphens.
  By default, message context is not shown, it has to be explicitly enabled by giving the :option:`cmake --log-context` command-line option or by setting the :variable:`CMAKE_MESSAGE_CONTEXT_SHOW` variable to true.
  See the :variable:`CMAKE_MESSAGE_CONTEXT` documentation for usage examples.

CMake Warning and Error message text displays using a simple markup language.
Non-indented text is formatted in line-wrapped paragraphs delimited by newlines.
Indented text is considered pre-formatted.


チェックした結果を報告するメッセージ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.17

A common pattern in CMake output is a message indicating the start of some
sort of check, followed by another message reporting the result of that check.
For example:

.. code-block:: cmake

  message(STATUS "Looking for someheader.h")
  #... do the checks, set checkSuccess with the result
  if(checkSuccess)
    message(STATUS "Looking for someheader.h - found")
  else()
    message(STATUS "Looking for someheader.h - not found")
  endif()

This can be more robustly and conveniently expressed using the ``CHECK_...``
keyword form of the ``message()`` command:

.. code-block:: cmake

  message(<checkState> "message" ...)

where ``<checkState>`` must be one of the following:

  ``CHECK_START``
    Record a concise message about the check about to be performed.

  ``CHECK_PASS``
    Record a successful result for a check.

  ``CHECK_FAIL``
    Record an unsuccessful result for a check.

When recording a check result, the command repeats the message from the most
recently started check for which no result has yet been reported, then some
separator characters and then the message text provided after the
``CHECK_PASS`` or ``CHECK_FAIL`` keyword.  Check messages are always reported
at ``STATUS`` log level.

Checks may be nested and every ``CHECK_START`` should have exactly one
matching ``CHECK_PASS`` or ``CHECK_FAIL``.
The :variable:`CMAKE_MESSAGE_INDENT` variable can also be used to add
indenting to nested checks if desired.  For example:

.. code-block:: cmake

  message(CHECK_START "Finding my things")
  list(APPEND CMAKE_MESSAGE_INDENT "  ")
  unset(missingComponents)

  message(CHECK_START "Finding partA")
  # ... do check, assume we find A
  message(CHECK_PASS "found")

  message(CHECK_START "Finding partB")
  # ... do check, assume we don't find B
  list(APPEND missingComponents B)
  message(CHECK_FAIL "not found")

  list(POP_BACK CMAKE_MESSAGE_INDENT)
  if(missingComponents)
    message(CHECK_FAIL "missing components: ${missingComponents}")
  else()
    message(CHECK_PASS "all components found")
  endif()

Output from the above would appear something like the following::

  -- Finding my things
  --   Finding partA
  --   Finding partA - found
  --   Finding partB
  --   Finding partB - not found
  -- Finding my things - missing components: B

Configure ログのメッセージ
^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 3.26

.. code-block:: cmake

  message(CONFIGURE_LOG <text>...)

Record a :ref:`configure-log message event <message configure-log event>`
with the specified ``<text>``.  By convention, if the text contains more
than one line, the first line should be a summary of the event.

This mode is intended to record the details of a system inspection check
or other one-time operation guarded by a cache entry, but that is not
performed using :command:`try_compile` or :command:`try_run`, which
automatically log their details.  Projects should avoid calling it every
time CMake runs.  For example:

.. code-block:: cmake

  if (NOT DEFINED MY_CHECK_RESULT)
    # Print check summary in configure output.
    message(CHECK_START "My Check")

    # ... perform system inspection, e.g., with execute_process ...

    # Cache the result so we do not run the check again.
    set(MY_CHECK_RESULT "${MY_CHECK_RESULT}" CACHE INTERNAL "My Check")

    # Record the check details in the cmake-configure-log.
    message(CONFIGURE_LOG
      "My Check Result: ${MY_CHECK_RESULT}\n"
      "${details}"
    )

    # Print check result in configure output.
    if(MY_CHECK_RESULT)
      message(CHECK_PASS "passed")
    else()
      message(CHECK_FAIL "failed")
    endif()
  endif()

If no project is currently being configured, such as in
:ref:`cmake -P <Script Processing Mode>` script mode,
this command does nothing.

参考情報
^^^^^^^^

* :command:`cmake_language(GET_MESSAGE_LOG_LEVEL)`
