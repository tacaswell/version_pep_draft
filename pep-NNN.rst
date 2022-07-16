PEP: <REQUIRED: pep number>
Title: <REQUIRED: pep title>
Author: <REQUIRED: list of authors' real names and optionally, email addrs>
Sponsor: <real name of sponsor>
PEP-Delegate: <PEP delegate's real name>
Discussions-To: <REQUIRED: URL of current canonical discussion thread>
Status: <REQUIRED: Draft | Active | Accepted | Provisional | Deferred | Rejected | Withdrawn | Final | Superseded>
Type: <REQUIRED: Standards Track | Informational | Process>
Content-Type: text/x-rst
Requires: <pep numbers>
Created: <date created on, in dd-mmm-yyyy format>
Python-Version: <version number>
Post-History: <REQUIRED: dates, in dd-mmm-yyyy format, and corresponding links to PEP discussion threads>
Replaces: <pep number>
Superseded-By: <pep number>
Resolution: <url>


Abstract
========

[A short (~200 word) description of the technical issue being addressed.]


Motivation
==========

[Clearly explain why the existing language specification is inadequate to address the problem that the PEP solves.]


In many contexts it useful to know the run-time version of an imported module.
This can be used both for attaching meta-data for provenance or for feature/bug
gating. While the pypa has made progress on defining and accessing package
versions for distributions and environment solving, run time versions have
different use cases.

Not every importable Python module is necessarily or can be installed with the
proper meta-data, but these modules still need access to run-time version
information.  Local modules (e.g. `.py` file next to an application) are
extremely useful as are other `$PYTHONPATH` manipulations. Additionally,
modules imported via the import hook machinery may lack the file structure of a
traditional package.  Given the richness of the import machinery in Python the
simplest way to determine what will be imported is to empirically check.  A
distribution may contain multiple top level modules.  Each module may have its
own distinct version, which may be decoupled from the distribution version.
Finally, while it is best practice to always use well defined environments,
being able verify the version of a currently imported module --- independent of
the package management system --- is invaluable for debugging and traceability.


We are standardizing the long-standing convention of attaching a `__version__`
attribute to the module to provide users access to the run time version of a
module.


Rationale
=========

[Describe why particular design decisions were made.]

We chose the name `__version__` for the attribute because there is a long
standing convention of using the attribute for this purpose.  There was
PEP 396 [0] from 2011 which was recently abandoned and is well established
in the Scientific Python community [maybe get numbers?].  The specification
in this PEP is a lightly adapted version of the text in PEP 396.

We propose to use PEP 440 for the version specification to be consistent with
the other version string in the Python ecosystem.


Specification
=============

[Describe the syntax and semantics of any new language feature.]

1. When a module (or package) includes a version number, the version SHOULD be
   available in the __version__ attribute.
2. For modules which live inside a namespace package, the module SHOULD include
   the __version__ attribute. The namespace package itself SHOULD NOT include
   its own __version__ attribute.
3. The __version__ attribute’s value MUST be a string.  Module version
   numbers SHOULD conform to the normalized version format specified in PEP 440.
4. Module version numbers MAY contain version control system supplied information or
   other semantically different version numbers (e.g. underlying library
   version number) consistent with PEP 440.
5. The __version__ attribute SHOULD be consistent with the importlib.metadata
   information.
6. Packages MAY dynamically compute the __version__ string at runtime.


Backwards Compatibility
=======================

[Describe potential impact and severity on pre-existing code.]

None.  This is already a well established convention

Security Implications
=====================

[How could a malicious user take advantage of this new feature?]

None.  If the attacker has the ability to change an attribute on a module at run time
they can any number of other malicious things.

How to Teach This
=================

[How to teach users, new and experienced, how to apply the PEP to their work.]

Q: I have done `import MyModule`, what is it's version?
A: `MyModule.__version__` !

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
importlib is solving a different problem.  Additionally accessing an attribute
is a very simple ergonomic API to access the version of a module currently in
your namespace.

Open Issues
===========

[Any points that are still being decided/discussed.]

Deferring any discussion of if modules should automatically fallback to importlib
when the user access `__version__` and it is not otherwise defined.

None

Footnotes
=========

[A collection of footnotes cited in the PEP, and a place to list non-inline hyperlink targets.]

[0] https://peps.python.org/pep-0396/

Copyright
=========

This document is placed in the public domain or under the
CC0-1.0-Universal license, whichever is more permissive.
