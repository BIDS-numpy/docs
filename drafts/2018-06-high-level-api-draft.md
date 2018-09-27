Written during NumPy sprint, 25 May 2018.  See also [roadmap](https://hackmd.io/ZZ9bTt0sRR2Aj3JCOMMYyA).

# Higher Level NumPy API

## Goals

- Standard API for producing and consuming NumPy's high level API
    - Path to complete coverage of full NumPy API
    - Like `__array_ufunc__` for close to the entire NumPy API
- Predictability
    - Know what is wrapped and what isn't
    - Duck arrays (e.g., dask and sparse) can opt out of implicit casting to ndarray
    - Backwards compatibility in NumPy itself
- Process goal: needs to support a faster pace of innovation than is currently possible in NumPy.
    - Separate repo for development
    - Write the overloads in Python, not C
- Python 3 only 

## Use cases

- API users
    - e.g., xarray, dask, units
- API implementors
    - e.g., dask, units, sparse, cupy
- Objects that only implement `__array__`, but don't implement NumPy's API
    - e.g., patsy, pandas

## (Outdated) Proposal

- Experimental `numpy.api` module
    - Separate repo / project
        - Numpy-API depends on NumPy, and NumPy depends on NumPy
        - This will required coordinated releases, like dask/dask-distributed
        - Developed at github.com/numpy/numpy-api
    - Minimal requirements for NEPs
    - A strategy for implementing overloads:
        1. New protocols:
            - Protocol for connecting to duck arrays
            - Protocols for overloading arbitrary NumPy functions
        2. And/or dispatching:
            - Requires less coordination, because implementations can be written in a separate module rather than on the array objects directly.

## Draft NEP

https://hackmd.io/OhfNbBIPRXaC1fvrjQ3Grw

## ? Examples / use cases

```python
import numpy.api as np

np.sin(np.array(X))
np.ptp(sparse_arr)
```

- Everything can be overloaded
- No implicit casting to ndarray

