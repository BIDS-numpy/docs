# NumPy sprint: May 10–11 2019
Attending locally: Stefan, Tyler, Matti, Sebastian, Stephan H
Arriving: Eric (9:50 PM, WN2278 from SEA), Chuck (when?), Hameer (9:55 PM Thursday, UA-413 from Cleveland)

## Summary (written May 13)

Eight NumPy developers met May 10/11 to discuss design decisions, particularly around two topics: a new design for data types, and the API of the recently rewritten random number generation module. For discussions such as these that involve debating various technical intricacies, the high bandwidth of face-to-face meetings is invaluable. We thank the developers who generously gave their time to take part in these discussions, and can happily report progress on the following fronts:

##### Dtypes
This long discussion led to a deeper understanding of the overall design.
While design decisions may change we have made progress allowing to start to
work on a prototype implementation.
- Cleanup
  - We reached consensus on a refactor of the PyArray_Descr TypeObject, in a way that will be backward compatible
  - Create a new function to return a new PyArray_Descr object, hiding the actual layout of this object from the public API.
  - Deprecate using the current registration system for new dtypes, which will encourage users to use the new function.
  - Then we are free to refactor the `ArrFuncs` (legacy functions that live on the dtype)
- We discussed casting and promotion. There are still open questions around promotion:
  - What to do for "common type" operations like concatenate. The promotion rules are not dependent on the function.
  - Ufuncs should have a loop specific type resolver
  - Do we need a general registration for promotion or can we use some method on the dtype to resolve this?
      - We do require more than current safe-cast logic
      - registration may not be necessary (but we tended towards this)
  - Casting could live on the dtypes, or be handled much like a ufunc
  - We want to deprecate value-based promotion in the `result_type` logic,
    this would remove the need for complex work-arounds to remain backward
    compatible in `result_type` ("common type").
- Ufuncs may be refactored into:
  1. A (multiple) dispatched type resolver step based on category of types:
      - Most numpy types would be in the same category, but user
        different user types would be dispatched.
      - The type resolver would define the exact output dtypes as well
        as return a `UfuncImpl` object (or raise an error)
          - A user dtype type resolver, could call the 
      - Numpy could cache the two steps for optimization
  2. Much of the current `UFunc` logic would need to be moved into
     the `UFuncImpl`. Although, most of it could be inherited/default.
      - Multiple `UFuncImpl` classes could exist, allowing for
        much flexibility to support old numpy behaviour
      - `UFuncImpl` would probably be allowed (but does not have to)
        handle the full iteration, the datatypes would be fixed.
          - There is still some discussion if this should be revised,
            but a first step may hide this detail.
      - May want Separate loops for contiguous/sse/avx/1d loops.
        An example for code which is currently optimized with many paths is
        the casting/copying code.
  3. The inner loop (for new style `UfuncImpl`):
      - should get a return value to indicate stop iteration
        (although this has lower priority)
      - we will not aim to make the signature more complex, the current
        data pointer is enough if setup/teardown is possible.
- DType as a python class: we will try to progress without metaclasses, and the use case for subclasses of dtypes is not clear

##### Randomgen
We had a helpful discussion via video call with Kevin Sheppard around remaining changes needed before merging randomgen. Subjects such as api exposure, names of the new objects, and what to do with the open/closed range integer uniform random number functions were all discussed and resolved.

##### Numpy 2.0
We should consider bundling several (small) breaking changes for a 2.0 release.  In general, we may consider moving a bit faster, while clearly conveying to the users that we will not make such changes arbitrarily and will commit to supporting mechanisms for backward compatibility.

##### `__skip_array_function__`
We adopted this name for the function wrapped by `__array_function__`. Stefan H. incorporated this into the NEP and issued a PR to expose it. He also found a nice way to improve the information emitted on an exception when calling an `__array_function__`

##### copying
- dtypes should be immutable (currently `name` can be changed)
- On `array.copy()` we want to compact structured array padding on copy (perhaps recursively, while respecting the dtype `align` flag. The `is_aligned_struct` flag is preserved by copy, and will respect the order of the field layout in memory. When creating a dtype via PEP 3118, we will set the `is_aligned_struct`. The solution for people want to keep padding is `array.astype(array.dtype)`. Aside: we may want to make sure we are correctly reflecting PEP 3118 aligment semantics, whatever they are.

##### General work on PRs and issues
In addition to all the discussions, we worked on 22 pull requests, merging 16 of them. This is above our usual pace of 3 pull requests per day.

---

## Fri May 10

### Morning

#### Dtypes

[Sebastian's document](https://hackmd.io/UVOtgj1wRZSsoNQCjkhq1g)
[Sebastian's slides](https://drive.google.com/file/d/1BXdeuCw45pLLOHN4Wd-skEzP-1JBjINC/view?usp=sharing)

##### Discussion

- Meta-discussions
  - Should scalars be merged with 0-d ndarrays? The basic conflict comes from the desire to view `array[0]` as both a python object and a reference to a piece of memory in an ndarray
  - Should scalars be instances of dtypes?
  - Should we have a base ndarray with no methods?

- What to do with copyswap, copyswapn?
  - Places inside numpy sometimes don't use these anyway 
- UFuncs
  - No meaningful way to pass data into inner loop of ufuncs
  - How to register ufunc loops?
  - Could we return a "loop object" that is the inner loop with some guards to check input satisfies shape, contiguous, dtype restrictions
  - [User level Python APIs](https://hackmd.io/6YmDt_PgSVORRNRxHyPaNQ) and multi-level dispatching
- Convert ArrFuncs to ufuncs (might need kwargs)? There will be many corner cases, as discovered when redoing `np.clip`
- Allow constructing ArrFuncs from python (for prototyping only), using the same approach of PyHeapType which wraps slots.
- Value-based promotion is problematic. Tensorflow only does value-based for Python objects.
- Agreement about a UniformObject array that has the same type for all the objects in the array
  - example: implement `np.add` with `nb_add` or other C slots 


There was an active discussion about how the new inner loop of ufuncs could be defined from python based on the User level Python APIs document
- Where does type resolution and output allocation happen?

Eric came up with a design (added to User Level Python APIs) for loop resolution. Led to a discussion of a loop dispatch mechanism. perhaps we define loops for contiguous, 1d, and general ndarrays

Topics to discuss:
- Maybe go through ufunc from start to finish
  - For use cases of categorical, units, ragged array, ... dtypes
- What to do with flexible dtypes
- Casting, promotion rules
- Ressurect the subtype PR and close the Metaclass PR
- Backward compatibility
  - Provide a function to allocate `PyArray_Descr`
  - Anyone using the current stack-allocated PyArray_Descr is supposed to call `Py*Register`, use this to mark it as *old*
- What is the python API for overriding/registering ArrFuncs
  - `PyArray_Result()` decays arrays into scalars which casts them into non-dtype things: allow passing a flag to out that will not decay

Discussion of Numpy 2.0 from the [wiki page](https://github.com/numpy/numpy/wiki/Backwards-incompatible-ideas-for-a-major-release)

Discussion of `__numpy_implementation__`

### Afternoon

3-4 call with Kevin Sheppard about random generator refactor (mattip lead)

https://github.com/numpy/numpy/issues/13164

[Concerns about new random APIs](https://hackmd.io/E68kqcDZSKyuhmlY0kK3Sg)

An interesting alternative design: [`jax.random`](https://github.com/google/jax#random-numbers-are-different)

##### Names
What should we call the `brng` and `RandomGenerator` classes
- brng can be Sampler or RandomSample: BitGenerator bit_generator
- `RandomGenerator` suggestions? np.random.Generator

##### Exposing APIs
Do we need to expose
- np.random.gen
  - Pro: easy to use
  - Con: anti-pattern
  - Change the module level functions in np.random.* to notice when a seed is set: not really a good idea.
  - Deprecate np.random.seed, add a np.random.legacy namespace. 
- np.random.gen.brng.generator 
  - Consensus: Remove this.

##### Multithreaded best practices
[doc](https://6868-908607-gh.circle-artifacts.com/0/home/circleci/repo/doc/build/html/reference/random/multithreading.html)

`rng.jump()` -> `rng.jumped()` which returns a new instance

https://github.com/google/jax#random-numbers-are-different

##### What to do with ...
- `random_integers` which should be deprecated? 
  - conclusion: 
    - Remove matlab-compatible `.rand` and `.randn`
    - Remove `random_integers`
    - `randint` -> integers, remove usemask (force to faster), add boolean`endpoint` kwarg with default False
    - `random_sample` -> `random` 
- Apache2 license of pcg32, pcg64

- "axis"-based kwarg to choice? something we should consider?
  - perhaps we should hold off on the axis idea for now; interaction with size is confusing? If both are specified, we should require size is an integer or raise.
  - Do whatever `take` does, and just point to its documentation
- check that the documentation of the default matches the brng default for Generator

## Sat May 11

### Morning

* Get core contributor PR list down
* Write some NEP skeletons in "working groups"
    * `dtype`/`ufunc` NEP
        * Required: Eric, Sebastian
    * Indexing NEP (almost done)
        * Required: Stephan
    * RandomGen NEP (almost done)
        * Required: Matti
    * Am I missing any?

### Afternoon

* `uarray` discussion (Hameer), if there's interest
    * Explanation of features/API.
    * Matti's comment: It's useless without a back-end
        * That could be changed...
    * Possible use-cases (I'm open to suggestions)
        * Related to duck-array discussion
        * Related to `ufunc`/`dtype` discussion
        * ...
    * Possible modifications to adapt to said use-cases
        * Uses drive development, not the other way around.
    * NEP if there is interest.

* NumPy 2.0 --- Path forward? (Hameer, Stephan, ...)

    * Question:
        * Should NumPy 2.0 just be like current NumPy releases, perhaps 1-2 years of changes coming in one release?
        * Or should NumPy 2.0 be willing to break backwards compatibility in a more serious way, e.g., by removing scalars. If we're willing to consider this, we might require writing `import numpy2`.
    * Targets:
        * Less than x% of people should be affected.
        * Things should become *easier*.
        * Should remove major maintenance bottlenecks
            * Like the ArrFuncs struct, ideally
    * When should we do this?
        * Set a target or defer indefinitely?
    * Do we want to change some of our policies for 2.0?
        * Right now the policy is basically a one loud voice can block progress.
    * Some successful examples
        * jinja2 and ggplot2 use a new namespace
        * Django 2.0 (Hameer)
    * Some bad examples
        * Python 2 -> 3 (Chuck)

## Topics

### Dtype planning
Required: Eric, Sebastian, Matti
- [PR](https://github.com/numpy/numpy/pull/12585) to make dtype a TypeObject
- [PR](https://github.com/numpy/numpy/pull/12630) for a dtype NEP
- [Dtype Brainstorming](https://github.com/numpy/numpy/wiki/Dtype-Brainstorming) from SciPy 2018
- [issue](https://github.com/numpy/numpy/issues/9474) about subtyping dtype in Pandas
- [random thoughts](https://hackmd.io/ok21UoAQQmOtSVk6keaJhw) by Matti about a high-level view
- Eric's [high-level view](https://hackmd.io/_TS-pb88RsCI9NEpOH3FXg?both)

### Random Number PR review
Required: Kevin Sheppard (remote), Robert Kern (remote) Matti, 
- [randomgen](https://github.com/bashtage/randomgen) where it all started
- [PR](https://github.com/numpy/numpy/pull/13163) to merge randomgen to NumPy
- [tracking issue](https://github.com/numpy/numpy/issues/13164) for things to be done before/after the merge
- Licensing. PCG is licensed under Apache 2. Robert expressed [concern](https://github.com/numpy/numpy/issues/13447#issuecomment-488547793) that this is incompatible with GPL2; [Apache website confirms](https://www.apache.org/licenses/GPL-compatibility.html) (Apache 2 is only compatible with GPL v3). PCG can therefore not be included.

### PR Review
Required: Eric, Tyler

### If there is time

- copying and views:
  - Does copy need to copy the dtype [issue](https://github.com/numpy/numpy/issues/12817) If not we should make dtype instances immutable and provide helpers to copy them
  - Does a copy compact memory or does it need a kwarg to force compacting?
    - Discussion and conclusion: on `array.copy()`we want to compact structured array padding on copy (perhaps recursively, respecting the dtype align flag. The is_aligned_struct flag is preseved by copy, and will respect the order of the field layout in memory. When creating a dtype via PEP 3118, we will set the is_aligned_struct.  The solution for people want to keep padding is `array.astype(array.dtype)`. Aside: we may want to make sure we are correctly reflecting PEP 3118 aligment semantics, whatever they are.

