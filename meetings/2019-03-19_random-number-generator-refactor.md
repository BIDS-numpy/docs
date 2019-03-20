# NumPy RNG refactor call

Present: Sebastian, Matti, Tyler, Stefan, Robert Kern, Kevin Sheppard, Hameer

## References

- [NEP 19](https://www.numpy.org/neps/nep-0019-rng-policy.html)
- [Kevin's proposed implementation](https://github.com/bashtage/randomgen)

## Notes

- `randomgen` build system integration with NumPy distutils: this may take a little effort to merge in
- This only needs to build on Python 3; we won't be backporting the RNG refactor
- Current work is strictly backward compatible, except for normal generator + dependencies; there is a LegacyGenerator that fully preserves backward stream compat
    - API may need to be pared back for instance do we support 32 bit generators?
    - Float casting: 64 -> 32 OK?
- Integration: Kevin wants randomgen to go into NumPy, instead of living as separate repo
- Currently there are many RNGs included; we may want to select a few for inclusion, conservatively
  - can have a separate repo for examples of additional random generators 
- MT19937 (32-bit) This generator **must** be included for backward compatibility. 
- [Xoroshiro](http://xoshiro.di.unimi.it/) would be a generator that should go in ; fast
    - RTK: I'm less a fan of these. [They seem to fail randomness tests early.](https://www.johndcook.com/blog/2017/08/14/testing-rngs-with-practrand/)
    - KS. The newer version seem to have addressed the shortcomings of these. They may have some issues with the lower bits but these are unlikely to affect most applications of uniforms since these used the upper 53.
- pcg64 is another but might be more difficult -- <I didn't hear why?>; fast
    - RTK: I'm more a fan of this one. There's something of a beef between the Vigna, the Xoroshiro author, and the PCG author. Vigna makes much of the fact that the PCG paper was rejected from ACM TOMS, but that does not appear to me to be a problem with the algorithm or even the paper, per se, just that it was not the right forum for the paper.
    - KS: Downside of PCG64 is that it is dog slow on Win32, and anything else that doesn't have first-class uint128 support. This might be enough to keep it form being the default basic generator, although I think should be included since it is so trivial to produce 1000s of independent streams and so could be used in the largest of cluster applications. 
- Intel hardware is an example of a generator that is frequently requested and could be optional
    - RTK: I suggest leaving this one to third party packages.
    - KS: Also should be 3rd party given lack of breadth of OS and CPU support.  It is, however, a good example of what *could* be done now, that was virtually impossible with the old design. I think it may even be possible to write basic RNGs in Numba, but this isn't important now. 

From NEP:
 
> In order to allow certain workarounds, it MUST be possible to replace the basic RNG underneath the global RandomState with any other basic RNG object

- Switching default backend is strongly discouraged.
- Example use case: An existing library is using `numpy.random` functions.  As a user, you cannot modify that library, but you want to use all the new distributions & RNGs.  This mechanism would be a way to install the new RNG behavior into the old/legacy `numpy.random` system.
- Discuss locking mechanism currently used---may need some improvements / adjustments
  - Would it make sense to add a slot in for a **lock** in the basic new generators?
- Easiest is not to blend the old system and the new
- Ideally, using the old `random` methods should advance the internal state of the new methods
    - Suggestion: joint seed modification
        - Watch out for correlation, even if using `(seed, seed + 1)`
    - Alternative: keep lock outside of instantiation of basic RNGs, and then use single internal state for both generators.
    - Seberg: Question: Are all generators thread safe, or would they need a second lock?
        - RTK: Only a single lock is necessary. It just needs to be acquired at the start of each distribution generator method and released at the end. The current implementation has the lock living on the distributions instance (i.e. the wrapper around `BasicRNG`) with the expectation that each distributions instance owns its own unique `BasicRNG` instance. My recommendation is to make the lock be owned by the `BasicRNG` instance such that the `BasicRNG` instances can be shared among distributions instances safely. The actual acquires/releases would still be in the same place.
- System entropy reading: should it be exposed?
- KS: Other API considerations that might need to be trimmed:
    - Repo attempts to make it less painful to directly produce scalar random variables in Cython by providing pxd files.  This has come up on GH a few times.
    - Similarly, distributions.c can be build into a dll/so now which allows it do be consumed by Numba or anything else that likes cffi.

## Kevin's previous email, for context

I think it is conceptually ready.  The basic design uses a "BasicRNG" to
provide some core functions and to retain state.  These are exposed to a
RandomGenerator object as opaque function pointers that return 64 bits, 32
bits or a double.  The reason for this design was to accommodate MT19937
(32 bits), most modern PRNG (64 bits) and dSFMT (doubles/53 bits).  The
BasicRNG retains all of the state information and provides methods to
get/set state and initialize.

Right now it slavishly passes the NumPy RandomState suite using MT19937
with one caveat -- the normals have been replaced with Ziggurat and so
distributions that rely on polar transforms (t, chi2, etc.) have been moved
to a module named `LegacyGenerator`.  This is very helpful to ensure
correctness.

I have consistently tested it on Win32/64, Linux 64, OSX and occasionally
on ARM (on a personal account on Scaleway).  If there are other platforms
it would be good to try these before sewing it in.  Most modern generators
are straight C and so simple.

The **big questions** I have are:

* How to review this before starting to sew it into NumPy?
* What should the names of the components be?  I lifted BasicRNG from
Intel's vector random generator for Python.  Also, RandomGenerator seemed
appropriate but this is a forever name so probably needs more consideration.
* What basic RNGs should be included?
* What novel functions should be dropped?  I've implemented some 32-bit
float generators and made a few other enhancements.  There has also been at
least 1 other user contribution that improves bounded uniform integers
using a modern method.
* I have a LegacyGenerator that ...

There are probably more issues but this is the 10-mile view.
