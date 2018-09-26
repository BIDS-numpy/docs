# Dtype tasks

Reading the issues, discussions, and brainstorming, here is my take on what we should do (rough draft), also see the [ufunc tutorial](http://www.numpy.org/devdocs/user/c-info.ufunc-tutorial.html?highlight=ufunc%20tutorial)

- [ ] Convert _PyArray_Descr into a PyTypeObject
  - will allow subclassing
  - requires refactoring the fields in the struct, which will break peoples compiled c-code (should be ok once they recompile)
  - People who want to add attributes like units and categories can do so from python or c by subclassing
  - Scalars - let's leave that for another refactoring. So we allow creating objects from this type (like today), they by definition are scalars.
  - Make sure the offset, elsize can be very large
- [ ] Add dunder methods to dtypes
  - Should they allow self? Probably not
  - needed for ufuncs
- [ ] Pass dtypes in to ufuncs
  - New inner loop signature, so we will need to support both old and new inner loops (don't mess with optimized stuff)
  - by default, inner loop will use dtype methods on arguments
- [ ] Allow subclassing dtype from both python and c
  - Uses dunder methods from python, they go to the slots 
