# Tech Note 014 - Interfaces for Oberon-2
### Oberon-2 type-bound procedures may be extended to support Interfaces a.k.a dynamic traits

Interfaces in the go programming language and dynamic traits in the rust programming language allow the programmer to factor functionality of separately compiled strongly-typed code for uniform access and composition at the cost of dynamically resolving at run-time the intersection of the procedures and functions of the interface and the procedures and functions of the record the interface resolves to.  Signatures not matching results in an error at run-time.

