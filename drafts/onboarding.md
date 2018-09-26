# Onboarding documentation for NumPy

Turn this into an issue so the plan is visible.  It will probably also generate some useful feedback.


## How to contribute to NumPy

Aim wide with this document: include as many potential developers, regardless of skill.

Ways in which you can contribute—see SciPy guide linked below.

1. Documentation
2. Website
3. Code review & issue triage
4. Code contribution
5. Helping others on StackOverflow

Also say things about domain experts being very valuable, etc. (https://github.com/scipy/scipy/issues/9030)

Why is it value to participate in, e.g., documentation. Good for the project, but also good for you: learning the code base, the overall structure, the tools involved.

> Documentation is often nice to get started with, if you want to focus on the process of contributing and don't want to worry too much about e.g. setting up for local development on your laptop. A valuable thing to do is to add nice examples to docstrings for functions that don't yet have an example. See #7168 for an overview. — Ralf for SciPy2018 (see issue 9030 linked above)

Emphasize value of contributing to non-coding tasks: web, copy editing, design, etc.  For coding: emphasize that we need reviews more than code.

Link out to a document for each type of contribution.  E.g., for web work, document how the website is built & how to use Sphinx to modify it.

Important: keep all pages SHORT and CONCISE.

## Technical guide: code contribution

This document should link out to the first.

When contributing code, please also review another PR.

- [Testing guidelines](https://docs.scipy.org/doc/numpy-1.15.0/reference/testing.html)

## Technical guide: documenting code

- [NumpyDoc](https://numpydoc.readthedocs.io/en/latest/)
- Steps to build documentation:
  Is this really the recommended way? maybe extend runtests.py to do this?
  ```
  pip install -e . scipy matplotlib # what about build log?
  cd doc; make clean; make html
  ```
- how does [this document](http://www.numpy.org/devdocs/docs/howto_document.html?highlight=numpy%20testing) relate to the numdoc guide?

## Technical guide: building and deploying numpy.org, docs.scipy.org

- How are the websites built? Describe the build/deploy mechanisms

### Concepts in NumPy

- [The missing guide to integers](https://github.com/scikit-learn/scikit-learn/wiki/C-integer-types:-the-missing-manual)
- Borrowed/owned references
- Deprecation mechanism
- API/ABI version number generation
- Templating (`*.src`)?
- Where to find different pieces of functionality (dtypes, ufuncs, built-in array methods, ...)

### Tools / tricks

- [Using Git](https://help.github.com/)
- Using valgrind, gdb, checking for buffer overruns
- Profiling and visualizing (kcachegrind?)

### FAQ

## Existing documentation
- [Contributing to Scipy](http://scipy.github.io/devdocs/hacking.html)
- [Overview](http://scipy.github.io/devdocs/index.html#developer-s-guide)
- [Numpy Workflow](http://www.numpy.org/devdocs/dev/gitwash/development_workflow.html) perhaps we should move this out to somewhere else

