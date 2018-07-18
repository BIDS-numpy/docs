# Numpy roadmap v2

*Note that this is meant as a roadmap only (and is still very much draft). We have in mind a hierarchy of documents/content here:*
- *at the highest level a single roadmap, containing only the most important NEPs and other important ideas (including non-code ones like "revamp numpy.org". This overview can live at http://www.numpy.org/neps/ but is not a NEP itself (more like an umbrella for them). Goal: a single page/slide/image that tells people what our big ticket items and overall direction are (plus perhaps a what's in/out of scope for NumPy, linked or included).*
- *then individual NEPs for single ideas.*
- *then a larger list of issues tagged with "wish list" that include ideas that are not yet written up as NEPs, and things that are too small for a NEP.*

*Furthermore there should be a document saying "this is what we plan to work on in the context of the NumPy grant".  That will include things from the roadmap, and also things like "triage issues, write better docs".*

Related content:

- Brainstorming from Berkeley / BOF discussion: https://hackmd.io/ZBvbHpwvRFKOeiLzpxrtaA
- Notes from the March 21, 2018 NEP sprint:
https://docs.google.com/document/d/10mmyZ2-9GDm4W_5xJIMnbSzxFrD55lJkNsH8F7UB_Fs/edit#
- Current "wish list" on the issue tracker: [link](https://github.com/numpy/numpy/issues?q=is%3Aopen+is%3Aissue+label%3A%2223+-+Wish+List%22)
Needs cleaning up, but a good place to collect ideas that are pre-NEP or not a good NEP fit.

## Scope of NumPy

Document overall what is in/out scope:

- In memory, N-dimensional single pointer + strided arrays on CPUs
    - Not specialized hardware
- Basic functionality for scientific computing
    - Overlapping functionality with SciPy:
        - linear algebra
        - FFT
        - Windowing functions for ffts.
        - where is the cutoff? Simple things that everyone needs should be in NumPy, specialized and complicated functions in SciPy. How do we decide which is which? 
    - We are happy with what we have.

    - Could potentially be split out into separate packages with coordinated releases:
        - random numbers
        - masked arrays
        - polynomials
        - financial?
    - Existing infrastructrure for data science packages (distutils, f2py, testing) 
- Higher level APIs for N-dimensional arrays
    - NumPy is a *de facto* standard for array APIs in Python
    - Protocols like `__array_ufunc__`.

### Things NumPy won't do

Labeled arrays:
- pandas and xarray have largely solved this problem
- Optional labels are worse than no labels at all (e.g., see [Never trust the row names of a dataframe in R
](https://www.perfectlyrandom.org/2015/06/16/never-trust-the-row-names-of-a-dataframe-in-R/))

GPU acceleration



## Wish list ideas
Rewriting internal functions as ufuncs (e.g., where, median, percentile, nan-reductions, sort, vectorize)

np.array() current accepts everything, but it should require dtype=object

Better dtypes:
- Easier custom dtypes
    - Simplify and/or wrap the current C-API
    - More consistent support for dtype metadata
    - Support for writing a dtype in Python
- New string dtype(s):
    - Fixed width encoded strings (utf8, latin1, ...)
    - Variable length strings (could share implementation with dtype=object, but are explicitly type-checked)
    - One of these should probably be the default for text data. The current behavior on Python 3 is neither efficient nor user friendly.
- np.int_ should not be platform dependent
- better coercion for string + number

Indexing:
- vindex/oindex [NEP 21](http://www.numpy.org/neps/nep-0021-advanced-indexing.html)

Duck typing
- Everything from [NEP 22](http://www.numpy.org/neps/nep-0022-ndarray-duck-typing-overview.html):
    - `np.asduckarray()`
- Mixins like `NDArrayOperatorsMixin`:
    - for mutable arrays
    - for reduction methods implemented as ufuncs

NumPy scalars
- The current implementation adds a large maintainence burden -- can we remove scalars and/or simplify it internally?
- Zero dimensional arrays get converted into scalars by most NumPy functions. Can this be fixed?

Typing
- Type annotations for NumPy: github.com/numpy/numpy-stubs
- Support for typing shape and dtype in multi-dimensional arrays in Python more generally

Speed improvements (document what kinds we want and don't want)

### Infrastructure (docs, CI, benchmarks, etc.)

Rewrite numpy.org

Run the benchmarks and render the resuts as part of the docs or website.

Continuous Integration:
- CI for more exotic platforms (e.g. ARM is now available from http://www.shippable.com/)
- Multi-package testing 
- Add an official channel for numpy dev builds for CI usage by other projects.

Figure out backwards compat & deprecation policy.  Specific examples to include in that policy:
matrix, MaskedArray, f2py, numpy.distutils, financial

Auto-build dev docs

Update steering council list etc.

### Functionality outside core

A backend for `numpy.fft` (so that e.g. `fft-mkl` doesn't need to monkeypatch numpy)

Write a strategy on how to deal with overlap between numpy and scipy for `linalg` and `fft` (and implement it).

`numpy.random` refactor (NEP 19)

Rewrite masked arrays to not be a ndarray subclass -- maybe in a separate project?
- MaskedArray as a duck-array type, and/or
- dtypes that support missing values

Deprecate `np.matrix`
