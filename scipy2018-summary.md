# Scipy 2018 summary

## NumPy intro at plenary session

## NumPy BOF

- We discussed using the roadmap as guideline.  Comments were made [here](https://hackmd.io/ZBvbHpwvRFKOeiLzpxrtaA).

### Roadmap

- [v2 document lives on the Wiki](https://github.com/numpy/numpy/wiki/numpy-roadmap-v2)
- Ralf wil be in BIDS next week, hopefully we can move it forward

## Numpy Sprint

- coverage -- PR to add Python-language unit test line coverage with pytest-cov results fed to codecov; adding % coverage badge to our landing README on github [to do: add in C-level line coverage with gcov, similar to SciPy]
- documentation - new contributor with 2 PRs
- AARCH64 (ARM 64 bit) now works [to do: add shippable "free"  or low-cost ARM node to CI testing]
- Travis Oliphant came, productive discussions
  - difference in philosophy between protocols and multimethods and subclassing: multimethods require registering and that NumPy sits in the middle, protocols can not include the numpy api at all
  - removing dtype methods to a basic minimum, all the rest should be ufuncs
  - what is a duck array, ideas on how it might work
- someone is going through and implementing a dtype - who was it?
- engaged with Oleksandr from Intel

## Dtype discussion

- [summarized in another document](https://github.com/numpy/numpy/wiki/Dtype-Brainstorming)
- [comments from the mailing list](https://mail.python.org/pipermail/numpy-discussion/2018-July/078423.html)

## For next year

- Lightning talk or full paper
- Tutorials? NumPy ? Wrapping C/C++ code (ctypes, cffi, cython, cppyy, pybind11, ?numba)?

## Development priorities

- ufuncs: matmul as a ufunc [Matti]
- Versioned deprecation warnings in NumPy [Stéfan]
- `loadtxt`: we'll pick off low-hanging fruit, and leave the rest to Pandas [Tyler]
- roadmap section for NEP website [Stéfan]
- version selector for NumPy docs [Stéfan]
- code coverage in C [Tyler]
- Develop a "doxygen" (or alternative, Pauli's tool?) generator from C to ref docs [Matti]
- Scan issues for other priorities [Matti & Tyler]
