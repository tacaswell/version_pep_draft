PEP: <REQUIRED: pep number>
Title: <REQUIRED: pep title>
Author: Thomas Caswell <tcaswell@gmail.com>, Christopher Barker <PythonCHB@gmail.com>
Sponsor: Michael Droettboom <mdboom@gmail.com>
PEP-Delegate: <PEP delegate's real name>
Discussions-To: <REQUIRED: URL of current canonical discussion thread>
Status: Draft
Type: Informational
Content-Type: text/x-rst
Requires: <pep numbers>
Created: <date created on, in dd-mmm-yyyy format>
Python-Version: <version number>
Post-History: <REQUIRED: dates, in dd-mmm-yyyy format, and corresponding links to PEP discussion threads>
Replaces: 396
Superseded-By: <pep number>
Resolution: <url>


[DRAFT NOTE:] I (CHB) am writing this from the perspective that all top-level importable modules(packages) *should* have a ``__version__`` attribute.

If that will not be accepted, then we can re-word it only slightly, such that it states that **if** a module author wants to have a version attribute, it *should* be called ``__version__``


Abstract
========

This PEP specifies that any imported module or package should have an attribute that provides the version of that module. That the attribute should be called ``__version__``, and the attribute should resolve to a string conforming to PEP 440 [2]_.

That is::

    import a_module
    a_module.__version__

should produce a PEP 440 compliant version string.

Motivation
==========

It is a long standing practice that software and modules should be able to self-report the version of the module itself. In the Python community, there have been various ways of providing this information.

A common approach has been to provide an attribute in the module (or package ``__init__.py``) that evaluates to the version string, such that ::

    import a_module
    a_module.__version__

evaluates to the version string of the imported module. Though a common practice, this approach has not been standardized, and there has been inconsistency as to the name of the attribute (e.g. ``.VERSION`` is fairly common as well). However, ``__version__`` is the most common name, and is a de-facto standard in the scientific software community.


More recently, the process for handling versions has been formalized and specified by PEPs [list here], so that the version specification of an *installed distribution package* is accessible via the standard library's `importlib.metadata` package, such that::

    import importlib.metadata
    importlib.metadata.version(distribution_name)

will evaluate to the version string of the installed distribution package.


Distribution Package vs. imported package:
------------------------------------------

The term "package" is overloaded in the Python community.

An "importable package" is a directory on the ``sys.path`` with a ``__init__.py`` file in it, and optionally other python modules or nested package directories. It can be imported via::

    import a_package

A "distribution package" ([2]_) is a collection of one or more python modules and packages that can be installed into a python system (usually via ``pip``). Once installed, the containing importable packages are available via the usual import mechanism.

In many cases, there is a one-to-one relationship between the distribution package and the installed importable package, e.g. ::

  $ python -m pip install a_package

then allows:

  >>> import a_package

and the version can be obtained via::

    import importlib.metadata
    importlib.metadata.version(a_package)

However, there is not always a simple one-to-one relationship between the distribution package and the importable package:

* The distribution package may have a different name than the importable package, e.g. ::

    $ python -m pip install beautifulsoup4

::

    >>> import bs4

And a distribution package may install more than one importable package.

In addition to these complications, in many contexts it useful to know the run-time version of an imported module. This can be used both for attaching meta-data for provenance or for feature/bug gating; run time version identification has different use cases than checking the version of installed distribution packages.

Fundamentally:

"what is the version of this installed distribution package?"

and

"what is the version of this imported module?" are two different, if often closely related, questions.

`importlib.metadata.version` is the standard way to obtain the version of a given installed distribution package.

This PEP proposes that the ``__version__`` attribute be the appropriate way to obtain the version of an imported package (module) at run time.

Alternative ways of making modules importable
---------------------------------------------

Not every importable Python module necessarily is, or can, be installed with the
proper meta-data.
But users of these modules may still need access to run-time version
information.
Local modules (e.g. ``*.py`` file next to an application) are
extremely useful, as are other ``sys.path`` manipulations.
Additionally, modules imported via the import hook machinery may lack the file structure of a traditional package.
Given the richness of the import machinery in Python the simplest way to determine what has been imported is to empirically check the imported module itself.

Finally, while it is best practice to always use well defined environments,
being able verify the version of a currently imported module --- independent of
the package management system --- is invaluable for debugging and traceability.

As an analogy: on Linux systems, software is generally installed via a package management system, e.g. yum or apt. When a user wants to know the version of an installed package, they can query the package management system, e.g. ::

  yum info git

However, if a user wants to know the version of a particular command line tool installed, they are most likely to query that tool itself, e.g. ::

  git --version

Because it answers the specific question they have: "which version am I running?", and because it may not be obvious what the name of the installed package is that command line tool was part of.


Rationale
=========

[Describe why particular design decisions were made.]

We chose the name ``__version__`` for the attribute because there is a long
standing convention of using the attribute for this purpose.  There was
PEP 396 [1]_ from 2011 which was recently abandoned and is well established
in the Scientific Python community [maybe get numbers?].  The specification
in this PEP is a lightly adapted version of the text in PEP 396.

We propose to use PEP 440 [2]_ for the version specification to be consistent with
the other version strings in use in the Python ecosystem.


Specification
=============

[Describe the syntax and semantics of any new language feature.]

1. When a module (or package) includes a version number, the version SHOULD be
   available in the ``__version__`` attribute of the top-level module.
2. For modules which live inside a namespace package, the module SHOULD include
   the ``__version__`` attribute. The namespace package itself SHOULD NOT include
   its own ``__version__`` attribute.
3. The ``__version__`` attribute’s value MUST be a string.  Module version
   numbers SHOULD conform to the normalized version format specified in PEP 440.
4. Module version numbers MAY contain version control system supplied information or
   other semantically different version numbers (e.g. underlying library
   version number) consistent with PEP 440.
5. The ``__version__`` attribute SHOULD be consistent with the ``importlib.metadata``
   information, where applicable.
6. Packages MAY dynamically compute the ``__version__`` string at runtime.

NOTE: this PEP does not propose the use of ``__version__`` in any standard library modules.

Backwards Compatibility
=======================

[Describe potential impact and severity on pre-existing code.]

This is already a well established convention -- there are three possible conditions of currently available packages:

1. It already has a ``__version__`` attribute: no impact.

2. It doesn't have a version attribute at all: these packages would need to add the attribute to be compliant, which would be fully backward compatible.

3. It has an attribute with a different name (e.g. ``VERSION``), or in a different location, such as a ``version.py`` file. In this case, the package would need to add a ``__version__`` attribute to be compliant, but could maintain an alias in the old name to ease the transition to the new format (and keep that alias indefinitely, if desired).


Security Implications
=====================

[How could a malicious user take advantage of this new feature?]

None.  If the attacker has the ability to change an attribute on a module at run time
they can do any number of other malicious things.

How to Teach This
=================

[How to teach users, new and experienced, how to apply the PEP to their work.]

Question: I have done ``import my_module``, what is its version?

Answer: ``my_module.__version__``.

Question: How do I make sure, when I build a distribution package, that the ``__version__`` attribute is correct, and in sync with what ``importlib.metadata.version`` will return?

Answer:

Option 1: Refer to your package build system's documentation: all the major systems support single-sourcing the version string (sometimes called dynamic metadata).

Option 2: put the line::

  __version__ = importlib.metadata.version("the_distribution_name")

in your module or packages ``__init__.py`` file.
Note that this option incurs a non-negligible module import cost, and requires that the packge be properly installed for it to work -- use with caution.

Reference Implementation
========================

[Link to any existing implementation and details about its state, e.g. proof-of-concept.]

None, no code changes to CPython.

Rejected Ideas
==============

[Why certain ideas that were brought while discussing this PEP were not
ultimately pursued.]

**relying solely on importlib**

As discussed above there are many cases where the metadata may not exist and
``importlib`` is solving a different problem.  Additionally accessing an attribute
is a very simple ergonomic API to access the version of a module currently in
your namespace.

Open Issues
===========

[Any points that are still being decided/discussed.]

Deferring any discussion of if modules should automatically fallback to ``importlib``
when the user access ``__version__`` and it is not otherwise defined.

Hmm -- alternatively, ``importlib`` could fall back to looking for ``__version__`` if the distribution isn't found. But I'll bet THAT would be even more controversial :-) - CHB

None

References
==========

[A collection of references cited in the PEP, and a place to list non-inline hyperlink targets.]

.. [1] PEP 396 - Module Version Numbers
   https://peps.python.org/pep-0396/

.. [2] PEP 440 - Version Identification and Dependency Specification
   https://peps.python.org/pep-0440/

.. [3] PR on the PyPa Packaging Users Guide
   https://github.com/pypa/packaging.python.org/pull/1580

.. [4] Recent Discussion in Discourse:
   https://discuss.python.org/t/please-make-package-version-go-away/58501

.. [5] Distribution Definition
   https://packaging.python.org/en/latest/glossary/#term-Distribution-Package

Copyright
=========

This document is placed in the public domain or under the
CC0-1.0-Universal license, whichever is more permissive.
