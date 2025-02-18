
pydeps
======

.. image:: https://readthedocs.org/projects/pydeps/badge/?version=latest
   :target: https://readthedocs.org/projects/pydeps/?badge=latest
   :alt: Documentation Status

.. image:: https://travis-ci.org/thebjorn/pydeps.svg
   :target: https://travis-ci.org/thebjorn/pydeps


.. image:: https://coveralls.io/repos/github/thebjorn/pydeps/badge.svg?branch=master
   :target: https://coveralls.io/github/thebjorn/pydeps?branch=master

.. image:: https://pepy.tech/badge/pydeps
   :target: https://pepy.tech/project/pydeps
   :alt: Downloads

Python module dependency visualization. This package installs the ``pydeps``
command, and normal usage will be to use it from the command line.

How to install
==============
::

    pip install pydeps

Basic Usage
-----------
From the shell::

    shell> pydeps [flags] module-directory

Detailed usage examples can be found below the version history.

**Creating the graph:**

To create graphs you need to install Graphviz_ Please follow the
installation instructions provided in the Graphviz link (and make
sure the ``dot`` command is on your path).

**Displaying the graph:**


To display the resulting `.svg` files, ``pydeps`` by default
calls ``firefox foo.svg``.  This is can be overridden with
the ``--display PROGRAM`` option, where ``PROGRAM`` is an
executable that can display the image file of the graph.

**Feature requests and bug reports:**

Please report bugs and feature requests on GitHub at
https://github.com/thebjorn/pydeps/issues

Version history
---------------

**Version 1.8.2** incldes a new flag ``--only`` that causes pydeps to 
only report on the paths specified::

    shell> pydeps mypackage --only mypackage.a mypackage.b

**Version 1.8.0** includes 4 new flags for drawing external dependencies as
clusters. See below for examples.
Additionally, the arrowheads now have the color of the source node.

**Version 1.7.3** includes a new flag ``-xx`` or ``--exclude-exact`` which
matches the functionality of the ``--exclude`` flag, except it requires an
exact match, i.e. ``-xx foo.bar`` will exclude foo.bar, but not
``foo.bar.blob`` (thanks to AvenzaOleg_ for the PR).

**Version 1.7.2** includes a new flag, ``--no-output``, which prevents
creation of the .svg/.png file.

**Version 1.7.1** fixes excludes in .pydeps files (thanks to eqvis_
for the bug report).

**Version 1.7.0** The new ``--reverse`` flag reverses the direction
of the arrows in the dependency graph, so they point _to_ the imported
module instead of _from_ the imported module (thanks to goetzk_ for
the bug report and tobiasmaier_ for the PR!).

**Version 1.5.0** Python 3 support (thanks to eight04_ for the PR).

**Version 1.3.4** ``--externals`` will now include modules that 
haven't been installed (what ``modulefinder`` calls ``badmodules``).

**Version 1.2.8** A shortcut for finding the direct external dependencies
of a package was added::

    pydeps --externals mypackage

which will print a json formatted list of module names to the screen, e.g.::

    (dev) go|c:\srv\lib\dk-tasklib> pydeps --externals dktasklib
    [
        "dkfileutils"
    ]

which means that the ``dktasklib`` package only depends on the ``dkfileutils``
package.

This functionality is also available programmatically::

    import os
    from pydeps.pydeps import externals
    # the directory that contains setup.py (one level up from actual package):
    os.chdir('package-directory')
    print externals('mypackage')

**Version 1.2.5:** The defaults are now sensible, such that::

    shell> pydeps mypackage

will likely do what you want. It is the same as
``pydeps --show --max-bacon=2 mypackage`` which means display the
dependency graph in your browser, but limit it to two hops (which
includes only the modules that your module imports -- not continuing
down the import chain).  The old default behavior is available with
``pydeps --noshow --max-bacon=0 mypackage``.



Usage
-----

This is the result of running ``pydeps`` on itself (``pydeps pydeps``):

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps.svg?sanitize=true

(full disclosure: this is for an early version of pydeps)

Bacon
~~~~~

``pydeps`` also contains an Erdős-like scoring function (a.k.a. Bacon
number, from Six degrees of Kevin Bacon
(http://en.wikipedia.org/wiki/Six_Degrees_of_Kevin_Bacon) that lets
you filter out modules that are more than a given number of 'hops'
away from the module you're interested in.  This is useful for finding
the interface a module has to the rest of the world.


To find pydeps' interface to the Python stdlib (less some very common
modules).

::

    shell> pydeps pydeps --show --max-bacon 2 --pylib -x os re types _* enum

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-pylib.svg?sanitize=true

``--max-bacon 2`` (the default) gives the modules that are at most 2
hops away, and modules that belong together have similar colors.
Compare that to the output with the ``--max-bacon=0`` (infinite)
filter:

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-pylib-all.svg?sanitize=true
   :width: 40%

.pydeps
-------

All options can also be set in a ``.pydeps`` file using ``.ini`` file
syntax (parsable by ``ConfigParser``). Command line options override
options in the ``.pydeps`` file in the current directory, which again
overrides options in the user's home directory
(``%USERPROFILE%\.pydeps`` on Windows and ``${HOME}/.pydeps``
otherwise).

An example .pydeps file::

    [pydeps]
    max_bacon = 2
    verbose = 0
    pylib = False
    exclude =
        os
        re
        sys
        collections
        __future__



Import cycles
-------------

``pydeps`` can detect and display cycles with the ``--show-cycles``
parameter.  This will _only_ display the cycles, and for big libraries
it is not a particularly fast operation.  Given a folder with the
following contents (this uses yaml to define a directory structure,
like in the tests)::

        relimp:
            - __init__.py
            - a.py: |
                from . import b
            - b.py: |
                from . import a

``pydeps relimp --show-cycles`` displays:

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-cycle.svg?sanitize=true

Clustering externals
--------------------

Running `pydeps pydeps --max-bacon=4` on version 1.8.0 of pydeps gives the following graph:

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-18-bacon4.svg?sanitize=true

If you are not interested in the internal structure of external modules, you can add the ``--cluster`` flag, which
will collapse external modules into folder-shaped objects::

    shell> pydeps pydeps --max-bacon=4 --cluster

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-18-bacon4-cluster.svg?sanitize=true

To see the internal structure _and_ delineate external modules, use the ``--max-cluster-size`` flag, which controls
how many nodes can be in a cluster before it is collapsed to a folder icon::

    shell> pydeps pydeps --max-bacon=4 --cluster --max-cluster-size=1000

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-18-bacon4-cluster-max1000.svg?sanitize=true

or, using a smaller max-cluster-size::

    shell> pydeps pydeps --max-bacon=4 --cluster --max-cluster-size=3

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-18-bacon4-cluster-max3.svg?sanitize=true

To remove clusters with too few nodes, use the ``--min-cluster-size`` flag::

    shell> pydeps pydeps --max-bacon=4 --cluster --max-cluster-size=3 --min-cluster-size=2

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-18-bacon4-cluster-max3-min2.svg?sanitize=true

In some situations it can be useful to draw the target module as a cluster::

    shell> pydeps pydeps --max-bacon=4 --cluster --max-cluster-size=3 --min-cluster-size=2 --keep-target-cluster

.. image:: https://raw.githubusercontent.com/thebjorn/pydeps/master/docs/_static/pydeps-18-bacon4-cluster-max3-min2-keep-target.svg?sanitize=true


Intermediate format
-------------------

An attempt has been made to keep the intermediate formats readable,
eg. the output from ``pydeps --show-deps ..`` looks like this::

    ...
    "pydeps.mf27": {
        "imported_by": [
            "__main__",
            "pydeps.py2depgraph"
        ],
        "kind": "imp.PY_SOURCE",
        "name": "pydeps.mf27",
        "path": "pydeps\\mf27.py"
    },
    "pydeps.py2depgraph": {
        "imported_by": [
            "__main__",
            "pydeps.pydeps"
        ],
        "imports": [
            "pydeps.depgraph",
            "pydeps.mf27"
        ],
        "kind": "imp.PY_SOURCE",
        "name": "pydeps.py2depgraph",
        "path": "pydeps\\py2depgraph.py"
    }, ...

Usage (parameters)
------------------
::

    usage: pydeps [-h] [--debug] [--config FILE] [--no-config] [--version]
                  [-L LOG] [-v] [-o file] [-T FORMAT] [--display PROGRAM]
                  [--noshow] [--show-deps] [--show-raw-deps] [--show-dot]
                  [--nodot] [--no-output] [--show-cycles] [--debug-mf INT]
                  [--noise-level INT] [--max-bacon INT] [--pylib] [--pylib-all]
                  [--include-missing] [-x PATTERN [PATTERN ...]]
                  [-xx MODULE [MODULE ...]] [--externals] [--reverse] [--cluster]
                  [--min-cluster-size INT] [--max-cluster-size INT]
                  [--keep-target-cluster]
                  fname

positional arguments:
  fname                 filename

optional arguments:
  -h, --help                             show this help message and exit
  --config FILE                          specify config file
  --no-config                            disable processing of config files
  --version                              print pydeps version
  -L LOG, --log LOG                      set log-level to one of CRITICAL, ERROR, WARNING, INFO, DEBUG, NOTSET.
  -v, --verbose                          be more verbose (-vv, -vvv for more verbosity)
  -o file                                write output to 'file'
  -T FORMAT                              output format (svg|png)
  --display PROGRAM                      program to use to display the graph (png or svg file depending on the T parameter)
  --noshow                               don't call external program to display graph
  --show-deps                            show output of dependency analysis
  --show-raw-deps                        show output of dependency analysis before removing skips
  --show-dot                             show output of dot conversion
  --nodot                                skip dot conversion
  --no-output                            don't create .svg/.png file, implies --no-show (-t/-o will be ignored)
  --show-cycles                          show only import cycles
  --debug                                turn on all the show and verbose options (mainly for debugging pydeps itself)
  --noise-level INT                      exclude sources or sinks with degree greater than noise-level
  --max-bacon INT                        exclude nodes that are more than n hops away (default=2, 0 -> infinite)
  --pylib                                include python std lib modules
  --pylib-all                            include python all std lib modules (incl. C modules)
  --x PATTERN, --exclude PATTERN         input files to skip (e.g. `foo.*`), multiple patterns can be provided
  --xx MODULE, --exclude-exact MODULE    same as --exclude, except requires the full match. `-xx foo.bar` will exclude foo.bar, but not foo.bar.blob
  --only MODULE_PATH                     only include modules that start with MODULE_PATH, multiple paths can be provided
  --externals                            create list of direct external dependencies
  --reverse                              draw arrows to (instead of from) imported modules
  --cluster                              draw external dependencies as separate clusters
  --min-cluster-size INT                 the minimum number of nodes a dependency must have before being clustered (default=0)
  --max-cluster-size INT                 the maximum number of nodes a dependency can have before the cluster is collapsed to a single node (default=0)
  --keep-target-cluster                  draw target module as a cluster


     
You can of course import ``pydeps`` from Python (look in the
``tests/test_relative_imports.py`` file for examples.

Contributing
------------
#. Fork it
#. It is appreciated (but not required) if you raise an issue first: https://github.com/thebjorn/pydeps/issues
#. Create your feature branch (`git checkout -b my-new-feature`)
#. Commit your changes (`git commit -am 'Add some feature'`)
#. Push to the branch (`git push origin my-new-feature`)
#. Create new Pull Request


.. _Graphviz: http://www.graphviz.org/download/
.. _AvenzaOleg: https://github.com/avenzaoleg
.. _eqvis: https://github.com/eqvis
.. _goetzk: https://github.com/goetzk
.. _tobiasmaier: https://github.com/tobiasmaier
.. _eight04: https://github.com/eight04
