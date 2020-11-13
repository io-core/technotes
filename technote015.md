# Tech Note 015 - DYN variables for dynamic scope in Oberon
### Provide dynamic scope via DYN variables to provide better error handling in Oberon

* Oberon variables are statically scoped
* Dynamic scope can be emulated by:
 * saving a referenced global variable on the stack upon procedure entry
 * accessing and modifying the global variable as usual
 * restoring the cached value upon procedure exit
* DYN variables may enable the common lisp condition system in Oberon
* This kind of dynamic scope may only work local to the thread of execution
* This kind of dynamic scope is ineficient for large objects

