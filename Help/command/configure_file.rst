configure_file
--------------

ファイルを別の場所にコピーして、その内容を変更する。

.. code-block:: cmake

  configure_file(<input> <output>
                 [NO_SOURCE_PERMISSIONS | USE_SOURCE_PERMISSIONS |
                  FILE_PERMISSIONS <permissions>...]
                 [COPYONLY] [ESCAPE_QUOTES] [@ONLY]
                 [NEWLINE_STYLE [UNIX|DOS|WIN32|LF|CRLF] ])

``<input>`` ファイルを ``<output>`` ファイルにコピーし、そのファイルの中にある ``@VAR@``、``${VAR}``、``$CACHE{VAR}``、``$ENV{VAR}`` として参照できる変数の値を置き換えます。
変数の参照部分はそれぞれ、その変数が現在保持している値で置き換えられるか、またはその変数が定義されていなかったら空の文字列で置き換えられます。
さらに、次のような書式を持つ行は

.. code-block:: c

  #cmakedefine VAR ...

次のいずれかで置き換えられます：

.. code-block:: c

  #define VAR ...

または

.. code-block:: c

  /* #undef VAR */

どちらで置き換えられるかは ``VAR`` という変数に何がセットされているかに依存します（たとえば :command:`if` コマンドで ``VAR`` を判定した結果）。
また、変数名につづく "``...``" の部分に何か指定されている場合は、``VAR`` が "``...``" の文字列で置き換えられます。
さらに ``#cmakedefine01 VAR`` の書式を持つ行は ``VAR`` 自身が ``VAR 0`` または ``VAR 1`` に展開されます。
したがって、次のような書式を持つ行は

.. code-block:: c

  #cmakedefine01 VAR

次のいずれかで置き換えられます：

.. code-block:: c

  #define VAR 0

または

.. code-block:: c

  #define VAR 1

Input lines of the form ``#cmakedefine01 VAR ...`` will expand as ``#cmakedefine01 VAR ... 0`` or ``#cmakedefine01 VAR ... 1``, which may lead to undefined behavior.

.. versionadded:: 3.10
  The result lines (with the exception of the ``#undef`` comments) can be indented using spaces and/or tabs between the ``#`` character and the ``cmakedefine`` or ``cmakedefine01`` words.
  This whitespace indentation will be preserved in the output lines:

  .. code-block:: c

    #  cmakedefine VAR
    #  cmakedefine01 VAR

  will be replaced, if ``VAR`` is defined, with

  .. code-block:: c

    #  define VAR
    #  define VAR 1

If the input file is modified the build system will re-run CMake to
re-configure the file and generate the build system again.
The generated file is modified and its timestamp updated on subsequent
cmake runs only if its content is changed.

The arguments are:

``<input>``
  Path to the input file.  A relative path is treated with respect to
  the value of :variable:`CMAKE_CURRENT_SOURCE_DIR`.  The input path
  must be a file, not a directory.

``<output>``
  Path to the output file or directory.  A relative path is treated
  with respect to the value of :variable:`CMAKE_CURRENT_BINARY_DIR`.
  If the path names an existing directory the output file is placed
  in that directory with the same file name as the input file.
  If the path contains non-existent directories, they are created.

``NO_SOURCE_PERMISSIONS``
  .. versionadded:: 3.19

  Do not transfer the permissions of the input file to the output file.
  The copied file permissions default to the standard 644 value
  (-rw-r--r--).

``USE_SOURCE_PERMISSIONS``
  .. versionadded:: 3.20

  Transfer the permissions of the input file to the output file.
  This is already the default behavior if none of the three permissions-related
  keywords are given (``NO_SOURCE_PERMISSIONS``, ``USE_SOURCE_PERMISSIONS``
  or ``FILE_PERMISSIONS``).  The ``USE_SOURCE_PERMISSIONS`` keyword mostly
  serves as a way of making the intended behavior clearer at the call site.

``FILE_PERMISSIONS <permissions>...``
  .. versionadded:: 3.20

  Ignore the input file's permissions and use the specified ``<permissions>``
  for the output file instead.

``COPYONLY``
  Copy the file without replacing any variable references or other
  content.  This option may not be used with ``NEWLINE_STYLE``.

``ESCAPE_QUOTES``
  Escape any substituted quotes with backslashes (C-style).

``@ONLY``
  Restrict variable replacement to references of the form ``@VAR@``.
  This is useful for configuring scripts that use ``${VAR}`` syntax.

``NEWLINE_STYLE <style>``
  Specify the newline style for the output file.  Specify
  ``UNIX`` or ``LF`` for ``\n`` newlines, or specify
  ``DOS``, ``WIN32``, or ``CRLF`` for ``\r\n`` newlines.
  This option may not be used with ``COPYONLY``.

例
^^

Consider a source tree containing a ``foo.h.in`` file:

.. code-block:: c

  #cmakedefine FOO_ENABLE
  #cmakedefine FOO_STRING "@FOO_STRING@"

An adjacent ``CMakeLists.txt`` may use ``configure_file`` to
configure the header:

.. code-block:: cmake

  option(FOO_ENABLE "Enable Foo" ON)
  if(FOO_ENABLE)
    set(FOO_STRING "foo")
  endif()
  configure_file(foo.h.in foo.h @ONLY)

This creates a ``foo.h`` in the build directory corresponding to
this source directory.  If the ``FOO_ENABLE`` option is on, the
configured file will contain:

.. code-block:: c

  #define FOO_ENABLE
  #define FOO_STRING "foo"

Otherwise it will contain:

.. code-block:: c

  /* #undef FOO_ENABLE */
  /* #undef FOO_STRING */

One may then use the :command:`target_include_directories` command to
specify the output directory as an include directory:

.. code-block:: cmake

  target_include_directories(<target> [SYSTEM] <INTERFACE|PUBLIC|PRIVATE> "${CMAKE_CURRENT_BINARY_DIR}")

so that sources may include the header as ``#include <foo.h>``.

参考情報
^^^^^^^^

* :command:`file(GENERATE)`
