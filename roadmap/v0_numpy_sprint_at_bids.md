# NumPy Roadmap (draft brainstorming)

This document lives at https://github.com/numpy/numpy/wiki/NumPy-Roadmap
It was created at the NumPy sprint held at BIDS, 24 May 2018.
Present during discussion: Charles Harris, Stephan Hoyer, Matti Picus, Jarrod Millman, Stéfan van der Walt, Nathaniel J Smith, Matthew Rocklin

Since then it has been added to by various authors. There was a [discussion](http://numpy-discussion.10968.n7.nabble.com/A-roadmap-for-NumPy-longer-term-planning-td45613.html) on the mailing list. We will revisit this at the upcoming [SciPy](https://scipy2018.scipy.org) conference at a BOF meeting.

## Links

- [numpy-grant-planning](https://github.com/njsmith/numpy-grant-planning) 

## Ongoing Maintenance Tasks

- triaging
- document both python and C-API (i.e. borrowed or new refs)
    - can we document reference borrowing close to C code (in the spirit of doctests for Python code)
- code coverage in tests
- policy for merging pull requests
  - bugfixes - does it fix the bug
  - releases have their own rules
  - at least one reviewer
  - documentation typos can be merged by the committer
  - enhancements should go through the mailing list
- actually deprecate things that are deprecated
- make internals private

## New functionality

- Extensible dtypes
- Overloads for duck arrays ([Stephan / Nathaniel's notes](https://docs.google.com/document/d/10mmyZ2-9GDm4W_5xJIMnbSzxFrD55lJkNsH8F7UB_Fs/edit?usp=sharing))
  - allow interop with dask, pytorch, cupy, xarray 
  - see Matthew Rocklin's [draft blog post](https://matthewrocklin.com/blog/work/2018/05/22/beyond-numpy) (dead link) and Stephan Hoyer's [response](https://gist.github.com/shoyer/bf6d2c7d271f05e08ca5f77d729a8917)
  - also see the [NEP](https://github.com/numpy/numpy/pull/11189) that grew out of this
- Random number generation refactor ([Robert's notes](https://gist.github.com/stefanv/8ab156ccf1a8d2355d96f131d3b079d6)) and allow backend switching
- [Typing stubs for mypy](https://github.com/numpy/numpy-stubs)
    - Basic support in NumPy
    - Standard way to write/check annotations for N-dimensional arrays in Python (this will need a PEP)
- **Internal refactorings** of almost-external things like MaskedArray, f2py, numpy.distutils, financial
- How to handle the **Matrix class**; we want to avoid new usage, reduce external dependency
- Better **configuration reporting** (use it when reporting bugs)
- Allow switching numpy.linalg backends

### Lessons learned / painful deprecations

We may not be able to change any of these Right Now.

- Currently strategy is: either pass to ufunc or convert to ndarray. Makes it hard to use NumPy as high-level API, because it is unknown (without reading the code) whether other classes will survive various operations. See discussion above.
- 0-dim arrays get converted into NumPy scalars

## Infrastructure / ecosystem

- [multibuild](https://github.com/matthew-brett/multibuild) wheel builder (Linux/OSX/Windows)
- Airspeed Speed Velocity for benchmarks (https://pv.github.io/numpy-bench/) or codespeed (speed.python.org)
- Maintain the websites scipy.org, numpy.org
- numpydoc
- pytest
    - To what extent should we use pytest APIs?
    - pytest is a little magical (e.g., fixtures)
    - Should define a philosophy. Discussion led to a direction of simple use of pytest, using as little magic as feasable.
- OpenBLAS

## Philosophy / scope

- In memory, N-dimensional single pointer + strided arrays on CPUs
    - Not specialized hardware
- Basic functionality for scientific computing
    - Could potentially be split out into separate packages with coordinated releases:
        - random numbers
        - masked arrays
        - polynomials
        - financial?
    - Overlapping functionality with SciPy:
        - linear algebra
        - FFT
        - Windowing functions for ffts.
        - where is the cutoff? Simple things that everyone needs should be in NumPy, specialized and complicated functions in SciPy. How do we decide which is which?
    - Infrastructrure for data science packages (distutils, f2py, testing)
- Higher level APIs for N-dimensional arrays
    - NumPy is a *de facto* standard for array APIs in Python
    - Protocols like `__array_ufunc__`.

## Longer term plans

These ideas may be used to solicit further funding.

- Missing values
- Labeled arrays
    - In or out?
    - To some extent solved by third-party libraries like xarray and pandas
    - Typing is also a potential solution
- Speed (probably out of scope, given current resources)
    - How important is this? To what extent should we compete with libraries like TensorFlow, etc.
    - Use intrinsics (AVX, SSE, ...)?
    - Establish a known benchmark suite which would serve to quantify the discussion. Any change would be measured against its effect across the entire suite.
    - JIT options
        - Rewriting internals in higher level language?
        - See also Travis's current efforts: [libxnd](https://github.com/plures/xnd) and [Plures](https://github.com/plures) more generally
        - Intermediate LLVM-like language for expressing array computation, can be shared across packages?

## Social

- Goal: grow number of core maintainers
    - Example (above) of documenting C code more carefully to lower barrier to entry
    - Challenge: retention; if we spend time training new contributors, how do we get them to stay 5 years instead of 1 or 2 (grad school, e.g.)
    - Engage with universities or industry (who could e.g. sponsor developer time)
        - Can we have a standard mechanisms whereby industry can engage
            - Sponsorship of: developer time / specific feature / ... ?
            - Could take on self-contained tasks, such as type stubs
            - Companies may benefit from smaller, better scoped packages; they may not want to take on "NumPy", but could be willing to engage with a smaller project that won't become a time sink. Hypothetical example: company like UK Met Office -> package like Masked Arrays
- Goal: more diverse and inclusive contributor community
    - Office hours for interested participants?
    - Sprints for beginners?
- Consideration: if we cannot move forward with features, external packages will work around us somehow. I.e., other packages in the ecosystem progress at faster rates than NumPy, if NumPy is unable to respond in a timely fashion to external needs, then ad-hoc infrastructure develops outside of the project, missing out on the benefits of NumPy's central place within the community.
    - It is also quite hard to get balanced input on this specific issue: members of community who want to move forward more quickly may not be around any more.
    - Examples:
        - Pandas, which used to rely on / couple more closely with NumPy.
        - https://github.com/dgasmith/opt_einsum
    
## Groups we may want to connect with

- Intel / MKL
- Tensorflow
- CuPy / Chainer
- Consumers: Autograd, Tangent, TensorLy, Optimized Einsum

## Notes

### Merge ratios

https://gist.github.com/stefanv/353ded6353435bd84fa5e0ab4443e7d5

Merge ratios for the past year:

```
1.00 Charles Harris
0.16 Eric Wieser
0.11 Allan Haldane
0.08 Marten van Kerkwijk
0.02 Julian Taylor
0.02 Sebastian Berg
0.02 Ralf Gommers
0.01 Jaime Fernandez
```

## Other things discussed during the sprint

- [Higher level API](https://hackmd.io/j5pmoftHRu--JfBaoiOTRA#) which became a [NEP](https://github.com/numpy/numpy/pull/11189)
