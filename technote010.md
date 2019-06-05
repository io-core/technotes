# Tech Note 010 - Communicating Sequential Processes in Oberon
### Communicating Sequential Processes without language modification

Implementing CSP in Oberon can be obtained with:

* Procedures as already exist in the Oberon language
* Procedure variables as already exist in the Oberon language
* Some new SYSTEM defined procedures (requiring cooperation of the compiler)
* Placing the stack(s) in the heap

Single-core CSP (concurrency) should not require large changes in the Oberon GC and Runtime.

Multi-core CSP (parallelism) might be approached by requiring any heap shared by cores to contain only immutable contents cooperatively allocated and garbage collected by the cores. This concept might be extended to cluster computing. 

Oberon Module binaries may be adapted for multicore by reqiring the global variables section to be padded to page alignment and separately mapped in the thread local store of each process to the same logical address but different, unique phyisical addresses.
