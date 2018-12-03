# Numpy Dev Meeting Nov 30 - Dec 1

This document lives at https://tinyurl.com/ybj4cqh2
Attending: Chuck, Eric, Matti, Tyler, Stéfan, Stephan

## Notes

### External NumPy-like C/C++ libraries

- Lots of existing efforts: [xtensor](https://github.com/QuantStack/xtensor) (xarray), [dynd](https://github.com/libdynd/dynd-python) (no longer active), Plures [xnd](https://github.com/plures/xnd), [libndtypes](https://github.com/plures/ndtypes) (Krahl, QuanSight)
    - How do we see NumPy fitting in with these?
        - How much of this can we get upstream?  Not that easy.
        - Would be challenging but great if we could take parts of NumPy and replace it with, e.g., XTensor or similar.
            - Power of XTensor comes with compilation. But NumPy is precompiled and shipped.  You lose a lot of the performance.
            - Some features like runtime datatypes may not even be supported.
        - Maybe xnd is a better fit?
        - Should ensure we have similar type representations across these libraries. Not any of these are supersets of NumPy: don't, e.g., support fields with overlapping offsets ``np.dtype(dict(names=['a', 'b'], offsets=[0, 1], formats=['i2', 'i2']))``.  So we can't just swap in.
            - We could potentially work around these minor cases (deprecate them?).
        - New project: Google [jax](https://github.com/google/jax), compiles into something that can be run on a GPU/TPU/CPU.
    - If we want a better dtype system/missing value approach/efficient masked arrays and someone's already written that, we could e.g. incorporate it.  But there would need to be some concrete advantage.
    - Make sure there's communication with these projects.

### Ufuncs
- Allow per-invocation state, such as preallocating temporary arrays to use across inner loop invocations.


### Data-types

- [dtype planning document](https://hackmd.io/cVdS9UyBRayZF-tIW1lC0g)
- Subclassing arrays vs using dtype types
    1. New stacking function works with every dtype
    2. New datatype, don't have to rewrite all array composition functions
- Dtype structs aren't hidden currently, so may be relied upon by some C code; so we'll likely introduce breaking changes.
- When subclassing arrays, your arrays often drop any meta-data you attach. AstroPy has to, essentially, add implementations of all NumPy functions.  The new function protocol helps make this easier, but doesn't  avoid all the work they need to do.
- We'll need to cast between dtypes
- Ufuncs
    - Sometimes we'll still need to implement custom loops: e.g., missing values; but other times the dtypes can do all the work.  I.e., two cases:
        - We want to write new low-level loops (e.g., for quaternion or masked dtypes). Requires writing C.
        - We just want to wrap existing loops (e.g., for units). Also: we want to write Python. Something like `__array_ufunc__` works well.
    - dtypes on loops could address most users' need.
    - dtypes need to register their own printing
    - scalar types should be instances of dtype types
    - array has an instantiation of the dtype type (``isinstance(arr.dtype, dtype) and isinstance(arr.dtype, type)`` is true)
    - Would we need to do NumPy 2.0, because we will break low-level C ABI? Changes may not be extensive enough to warrant this. (But it might make sense from a branding perspective.)
    - Conceptually, changes may be big enough to warrant that 2.0 label.
- Ragged arrays; not done by dtypes alone. Useful in domains like geometry.  But can often be handled in different ways, like tabular storage—a bit awkward. Do dtypes allocate memory for data?
- Dtypes needs some way to allocate/deallocate memory on arrays, e.g., so we can have an array of pointers pointing to other C objects. Use cases: variable length strings, ragged arrays, polygons. [gh-10721](https://github.com/numpy/numpy/issues/10721) shows a request for this.
- Dtype will be meta-class; array will have instance of that (a type).

- Use case prioritization:
    1. Writing a pure Python "wrapper" dtype
    2. Writing a new low-level dtype that does memory management with pointers

#### Steps

1. Turn `np.dtype` into a metaclass, so that instances of ``np.dtype`` are themselves `type`s.
    ```graphviz
    digraph G {
     { rank=same; object; "np.float64"; float; some_void; some_sub  }
     { rank=same; type; "np.dtype"; "np.subarray_dtype" }
      object -> type [color=red]
      "np.float64(1.0)" -> "np.float64" [label="instance of" color=red fontcolor=red]
      1.0 -> float [color=red]
      "np.float64" -> "np.dtype" [color=red]
      "np.float64" -> object
      "float" -> object  [label="subclasses"]
      float -> type [color=red]
      "np.dtype" -> type
      "np.subarray_dtype" -> "np.dtype"
      
      some_sub [ label="np.dtype(('u4', (2, 3)))"]
      some_void [label="np.dtype([('x', 'u4')])"]
      
      some_sub -> "np.subarray_dtype" [color=red]
      
      some_void -> object
      some_void -> "np.dtype" [color=red]
      "(1,)" -> some_void [color=red]
      
      # confusing builtin links
      type -> type [color=red tailport=ne headport=nw style=dashed]
      type -> object [ style=dashed]
      "np.dtype" -> type [color=red style=dashed]
    } 
    ```
   - Requirement for making scalars into instances; and `isinstance` works logically:
     ```python
     assert isinstance(arr.dtype, np.dtype)
     assert isinstance(arr[0], arr.dtype)
     assert isinstance(arr.dtype, type)  # implied by the above
     ```
   - Need dtypes to be metaclasses, because they can have extra information on them. Can map python class attributes to C struct properties, like `itemsize`. A related example [here](https://github.com/numpy/numpy/pull/12467/files#diff-1b5b71fee0ef8fd3ca22529af2a5cb8aR4274).
   - Allows dtypes to be written in Python.
   - 
2. Register printing; dtype tells us how to represent elements
3. Make scalars into instances of dtypes (that goes somewhere)
4. We need some way to override ufuncs from dtypes. Either:
      - Add methods to dtypes so they know which low-level loops to look-up, i.e., a lower level and less flexible version of `__array_ufunc__`. There are at least three things needed
         - What is the output dtype (casting)
         - What is the loop function that should be used
         - If there is no fast loop, give me an operation to do the low-level operation
    - or a more restrict variant of `__array_ufunc__` (only for high level Python dtypes) that restricts itself to not handle duckarrays:

        ```python
        class mydatetime(np.int64):
            @classmethod
            def __dtype_ufunc__(this_dtype, ufunc, args):
                args = [a.view(int64) for a in args]
                return ufunc(*args).view(dtype=this_dtype)  # will delegate to __array_ufunc__
        ```
        - Notes: should be called from *within* `numpy.ndarray.__array_ufunc__`.
        - Question: do we need a dtype version of `__array_function__`? Ideally not. There are some hard cases like overwriting `np.argmax` (not a ufunc but could be) and `np.mean` (not a ufunc and would require extra work to make it possible).
        - Question: does this happen before or after `__array_ufunc__`?

5. Go through the dtype object and figure out what needs to be there.

What should dtypes do:
- Currently: [`PyArray_ArrFuncs`](https://github.com/numpy/numpy/blob/b47ed76ebfbbd91b5d475eb19043a98e95dc2ad5/numpy/core/include/numpy/ndarraytypes.h#L444-L554)
    - Messy; a whole bunch of what's defined here no longer seems necessary?
    - eg `argmin`, `argmax` [would be better as `gufunc`s](https://trello.com/c/oM1XhxYv)
- Envisioned:
    - Printing
    - Get/Setitem
      - return a chunk of memory
      - turn that chunk of memory into a PyObject
    - Provides data used to calculate the offset to the memory
    - Overloading of ufuncs

[source for `PyUfuncObject`](https://github.com/numpy/numpy/blob/e34f5bb4e3aad2bb35fb6d9e3a5c1bfc85eb97eb/numpy/core/include/numpy/ufuncobject.h#L114-L229)

6. User defined ufuncs in Python? e.g., with `np.vectorize`?

### Function Dispatching

- [Benchmarks](https://github.com/numpy/numpy/pull/12317)
- How to access the fastest version?
    - `np.hstack.__no_dispatch__`, `.__implementation__`, `.__direct__` — we can bikeshed the name later :)
    - Cost is currently only about 5 µs, but the functions themselves also execute on that order

### Call with Travis Oliphant

- TO: Quansight Labs as a way to hire maintainers and collaboratively fund them
- TO: Lots of big companies spending money in the scientific computation space; developing NumPy-like products, often sharing the NumPy API due to user demand.  But they often don't participate in the conversation.  Potential for numpy etc. to become antiquated due to external pressures.
- SH: They care about performance and particular subsets of operations, less about flexibility.
- MP: Getting the data into the system, out of it and visualizing. That's where NumPy etc. still is very applicable.
- SH: But could see a situation in which they just switch to TensorFlow for everything.
- TO: Not all efforts out there are done well, otherwise would have been fine.  But be aware of those pockets of energies.  Current activity causes a lot of dispersion.
- TO: dtypes, numpy is trying to do better.
    - Intern at UChicago working on some of this, Anrud (?)
    - xnd work that is ongoing (originally part of blaze, dynd—Mark Wiebe, then Irwin Zaid); xnd—C library, simple libraries that can be broken apart and reused easily.  That NumPy can look at for inspiration, code, etc.
        - NumPy could, e.g., depend on live dtypes.
    - ndtypes has a very simple interface in Python.
    - Dtype object could be a different project.
    - Current is analogous to Python 1.  Every type is an instance of a class called dtype.
    - Created warts, such as the scalars.
    - Dtype should be a type (meta-type).
    - If you broke dtypes out into a separate project, people would not need to depend on NumPy to build new types.
        - Rest of Python world don't always need array computing, but they might need the dtypes.  Should use same type descriptors as we do. (MemoryView, e.g.)  Could be pushed into Python itself.  Even TensorFlow and similar could start to reuse the same dtypes.
    - Ufuncs should not live on dtypes.
        - Metadata, get/set.
- scalar objects need to instantiate from a  subtype of dtype with operations. 
  - How to handle instances of subarray types like `np.dtype((int, (3, 4)))`? 
- These are more `mtypes` — metadata types.  Which currently play the role of NumPy's dtypes, which may need to be richer objects.
    - NumPy could extend this mtype with operations, so that it can have ComplexFloat16, e.g.
- TO is pushing the idea of a dtype object that works across Python ecosystem.
- People:
    - Hameer
    - Stefan Krah (Berlin)
    - Chris Oustrich (undergrad intern, U Chicago)

- [Matti's dtype type PR](https://github.com/numpy/numpy/pull/12462) 
- Could deprecate np.Void
- Lots of discussion around: [`__dtype_ufunc__`](https://hackmd.io/6YmDt_PgSVORRNRxHyPaNQ)
- Something new **Ufunc chaining**: can we do this smarter? UFuncs already chain (cast then calculate then cast)
    - Torch/JITting—chains of functions

1. +1 for Metatype
2. Need to be able to hook Numba or some other JIT in

Can we turn ufunc.reduce into a higher level call to ufunc without having specific behaviour? `reducer(np.add)(x)` or `np.reduce(np.add, x)` instead of `np.add.reduce(x)`.
 * Can `reducer` return a gufunc?
   * No, `add.reduce` has simultaneous signatures `->`, `i->`, `ij->`, ..., depending on the value of the `axis `argument`

[The Mathematics of Arrays](https://arxiv.org/abs/0907.0796), Lenore Mullin

### Type Annotations

- Not a priority for the BIDS-numpy effort for now
- Could be a good year-long, funded project
    - Funding options: czi (Napari)?

### Diversity note

There are other ways in which we can enable diversity.  Make dtypes easy to build, show people how, let them loose and create a community of "plug-in builders" around NumPy.

## Topics for discussion (list constructed before the meeting)

- Update the roadmap
  - `__array_function__` - how are we doing?
    - Stephan Hoyer would like some help with the C version https://github.com/numpy/numpy/pull/12317. It has some problem with infinite recursion right now.
- Dtype refactoring
  - Discuss the [plan](https://hackmd.io/cVdS9UyBRayZF-tIW1lC0g?both#) and if it makes sense
  - Travis would like to take part, has some ideas
- If not already merged, get matmul in
- Community health and curation
  - Are the proper people on the proper Github/Azure/numpy.org teams and have access (lowering the bus factor)
  - Can we do more to encourage new users, more diversity?
  - Adding "topics" to our repos

