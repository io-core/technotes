# Tech Note 010 - Communicating Sequential Processes in Oberon
### Communicating Sequential Processes without language modification

Implementing CSP in Oberon can be obtained with:

* Procedures as already exist in the Oberon language
* Procedure variables as already exist in the Oberon language
* Some new SYSTEM defined procedures (requiring cooperation of the compiler)
* Placing the stack(s) in the heap

Single-core CSP (concurrency) should not require large changes in the Oberon GC and Runtime.

Multi-core CSP (parallelism) with a shared heap with mutable contents will requre further adatping the Oberon GC and Runtime.

Requiring the heap shared by cores to contain only immutable contents may limit the changes to the GC and Runtime required for implementing a parallel Oberon.
