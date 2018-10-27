
# Dtype tasks

Reading the issues, discussions, and [brainstorming](https://github.com/numpy/numpy/wiki/Dtype-Brainstorming), here is my (Matti's) take on what we should do (rough draft), also see the [ufunc tutorial](http://www.numpy.org/devdocs/user/c-info.ufunc-tutorial.html?highlight=ufunc%20tutorial)
- [ ] Use cases: 
  - units, 
  - enumerations (categories), 
  - datetime (refactor datetime64?), 
  - unique numerics like rational or fixed-point int/float,
  - quaternions, fixed-size data structures like multi-column data in Pandas
  - object types like str, unicode, shapely/GEOS geometries
  - missing data sentinal that inherit from the [24 numeric types](#Builtin-Numeric-types-by-number)
- [ ] Convert [_PyArray_Descr]() into a PyTypeObject
  - np.dtype('int32') will return the `PyDescr_Int32` **type object**
  - will allow subclassing dtypes
  - will break peoples compiled c-code (should be ok once they recompile)
  - Scalars - let's leave that for another refactoring. Eventually  allow creating objects from this type, that by definition are scalars.
  - Make sure the offset, elsize can be very large (today limited to int32)
- [ ] Add dunder methods to dtypes
  - Should they allow self? Probably not
  - needed for ufuncs. Replaces [`PyArray_SetNumericOps`](https://github.com/numpy/numpy/blob/v1.15.3/numpy/core/src/multiarray/number.c#L63)
  - Some (most?) of the ufuncs are already dunder methods in `PyNumberMethods`, some come from [`PyArray_ArrFuncs`]()
- [ ] Pass dtypes in to ufuncs
  - New inner loop signature, so we will need to support both old and new inner loops (don't mess with optimized stuff)
  - by default, inner loop will use dtype methods on arguments
  - except for current ufunc type selector mechanism for the 21 built-in dtypes
  - MKL can still override some of the 21 built-in float/int types with their own inner loop functions via [`PyUFunc_ReplaceLoopBySignature`](https://docs.scipy.org/doc/numpy/reference/c-api.ufunc.html#c.PyUFunc_ReplaceLoopBySignature) and ['PyUFunc_RegisterLoopForType'](https://docs.scipy.org/doc/numpy/user/c-info.beyond-basics.html#c.PyUFunc_RegisterLoopForType)
- [ ] Allow subclassing dtype from both python and c
  - Uses dunder methods from python, they go to the slots 
  - People who want to add attributes like units and categories can do so from python or c by subclassing
- [ ] How can a dtype cause a change in higher-level functions
  - Say I create two category dtypes and two arrays `a`,`b` from each. Then `a + b` means concatenation (or does it?). How can the dtype cause that to happen?
- [ ] Object-type dtypes like text, unicode, shapefiles. 
- [ ] What operations (slots) are required from a dtype? Is there a protocol like `__array_function__`? Does the default `np.dtype` simply return NotImplemented for all the slots?

### Builtin Numeric types by number
`>>> [' '.join([str(i), str(np.sctypeDict[i])[14:-2]]) for i in range(24)]`
`['0 bool_', '1 int8', '2 uint8', '3 int16', '4 uint16', '5 int32', '6 uint32', '7 int64', '8 uint64', '9 int64', '10 uint64', '11 float32', '12 float64', '13 float128', '14 complex64', '15 complex128', '16 complex256', '17 object_', '18 bytes_', '19 str_', '20 void', '21 datetime64', '22 timedelta64', '23 float16']`

