|PyPI version| |Build Status| |Coverage| |Commits since last|

.. |PyPI version| image:: https://badge.fury.io/py/vermin.svg
   :target: https://pypi.python.org/pypi/vermin/

.. |Build Status| image:: https://travis-ci.org/netromdk/vermin.svg?branch=master
   :target: https://travis-ci.org/netromdk/vermin

.. |Coverage| image:: https://coveralls.io/repos/github/netromdk/vermin/badge.svg?branch=master
   :target: https://coveralls.io/github/netromdk/vermin?branch=master

.. |Commits since last| image:: https://img.shields.io/github/commits-since/netromdk/vermin/latest.svg

Vermin
******

Concurrently detect the minimum Python versions needed to run code. Additionally, since the code is
vanilla Python, and it doesn't have any external dependencies, it works with v2.7+ and v3+.

It functions by parsing Python code into an abstract syntax tree (AST), which it traverses and
matches against internal dictionaries with 813 rules divided into 117 modules, 548
classes/functions/constants members of modules, 144 kwargs of functions, and 4 strftime directives.
Including looking for v2/v3 ``print expr`` and ``print(expr)``, ``long``, f-strings, boolean
constants, ``"..".format(..)``, imports (``import X``, ``from X import Y``, ``from X import *``),
function calls wrt. name and kwargs, ``strftime`` + ``strptime`` directives used, and function and
variable annotations. It tries to detect and ignore user-defined functions, classes, arguments, and
variables with names that clash with library-defined symbols.

Usage
=====

It is fairly straightforward to use Vermin::

  ./vermin.py /path/to/your/project

Or via `PyPi <https://pypi.python.org/pypi/vermin/>`__::

  % pip install vermin
  % vermin /path/to/your/project

When using continuous integration (CI) tools, like `Travis CI <https://travis-ci.org/>`_, Vermin can
be used to check that the minimum required versions didn't change. The following is an exerpt::

  install:
  - ./setup_virtual_env.sh
  - pip install vermin
  script:
  - vermin -t=2.7 -t=3 project_package otherfile.py

Examples
========

::

  % ./vermin.py
  Vermin 0.4.8
  Usage: ./vermin.py [options] <python source files and folders..>

  Options:
    -q      Quite mode. It only prints the final versions verdict.
    -v..    Verbosity level 1 to 3. -v, -vv, and -vvv shows increasingly more information.
            -v will show the individual versions required per file, -vv will additionally
            show which modules, functions etc. that constitutes the requirements.
    -t=V    Target version that files must abide by. Can be specified once or twice.
            If not met Vermin will exit with code 1.
    -p=N    Use N concurrent processes to analyze files (defaults to all cores = 8).
    -i      Ignore incompatible version warnings.
    -d      Dump AST node visits.

  % ./vermin.py -q vermin
  Minimum required versions: 2.7, 3.0

  % ./vermin.py -q -t=3.3 vermin
  Minimum required versions: 2.7, 3.0
  Target versions not met:   3.3
  % echo $?
  1

  % ./vermin.py -v examples
  Detecting python files..
  Analyzing 6 files using 8 processes..
               /path/to/examples/formatv2.py
  2.7, 3.2     /path/to/examples/argparse.py
  2.7, 3.0     /path/to/examples/formatv3.py
  2.0, 3.0     /path/to/examples/printv3.py
  !2, 3.4      /path/to/examples/abc.py
               /path/to/examples/unknown.py
  Minimum required versions:   3.4
  Incompatible versions:         2

  % ./vermin.py -vv /path/to/examples/abc.py
  Detecting python files..
  Analyzing using 8 processes..
  !2, 3.4      /path/to/examples/abc.py
    'abc' requires (2.6, 3.0)
    'abc.ABC' requires (None, 3.4)

  Minimum required versions: 3.4
  Incompatible versions:     2

Contributing
============

Contributions are very welcome, especially adding and updating detection rules of modules,
functions, classes etc. to cover as many Python versions as possible. For PRs, make sure to keep the
code vanilla Python and run ``make test`` first. Note that code must be remain valid and working on
Python v2.7+ and v3+.
