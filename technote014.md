# Tech Note 014 - Interfaces for Oberon-2
### Oberon-2 type-bound procedures may be extended to support Interfaces a.k.a dynamic traits

Interfaces in the go programming language and dynamic traits in the rust programming language allow the programmer to factor functionality of separately compiled strongly-typed code for uniform access and composition at the cost of dynamically resolving at run-time the intersection of the procedures and functions of the interface and the procedures and functions of the record the interface resolves to.  Signatures not matching results in an error at run-time.

```
  TYPE
       I* = POINTER TO IDesc;
       IDesc* = RECORD
            h: INTEGER
       END ;

       R* = POINTER TO RDesc;
       RDesc* = RECORD
            h: REAL
       END ;

       Stringer* = INTERFACE OF
            PROCEDURE String* (VAR a: ARRAY OF CHAR) ;
       END ;

  PROCEDURE ( i : I ) String* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "integer"
  END String;

  PROCEDURE ( r : R ) String* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "float"
  END String;

  PROCEDURE Test*;
      VAR i: I; r: t: ARRAY 32 OF CHAR;
        s: Stringer; 
  BEGIN
      NEW(i); NEW(r);

      s := i;  s.String( t );  
      s := r;  s.String( t );  
  END Test;

```


