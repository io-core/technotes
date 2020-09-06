# Tech Note 014 - Interfaces for Oberon-2
### Implementing Interfaces for Oberon-2 type-bound procedures

Oberon-2 extends the Oberon programming language to support type-bound procedures. Each record with associated type-bound procedures (a.k.a. its 'methods') has a pointer to a table that is used by calling code to locate the entry points of the methods. Typical object oriented programming in Oberon-2 leverages this arrangement.

Type safety in Oberon-2 requires pointers to only point to a specific base type or an extension of that base type of record. A pointer may not have any other type of record assigned to it even if the two base types are otherwise structurally or behaviorally compatible. 

It is sometimes useful to be able to refer to items that are implemented differently but which exhibit a compatible subset of behavior (i.e. have some methods with matching type signatures) but type safety should not be sacrificed for expediency. The 'go' programming language implements type-safe behavior matching via 'Interfaces' while the 'rust' programming language calls this 'dynamic traits.'

Some relatively small modifications to the Oberon-2 compiler and Oberon system enable Interfaces in Oberon.

To express Interfaces in an Oberon-2 program the syntax may be extended with a new keyword 'INTERFACE' and new syntax for declaring an Interface type, for example: 

```
MODULE W;
  TYPE
       Identifier* = INTERFACE OF
            PROCEDURE What* (VAR a: ARRAY OF CHAR) ;
       END 
END W.
```

Records with methods that implement an interface are not required to know of the existance of the interface beforehand:

```
MODULE T;
  TYPE
       I* = POINTER TO IDesc;
       IDesc* = RECORD
            h: INTEGER
       END ;

       R* = POINTER TO RDesc;
       RDesc* = RECORD
            h: REAL
       END ;

  PROCEDURE ( i : I ) What* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "integer"
  END What;

  PROCEDURE ( r : R ) What* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "float"
  END What;
  
END T.  
```

Different types in different modules may implement the same interface:

```
MODULE GPS;
  TYPE
       LOC* = POINTER TO LOCDesc;
       LOCDesc* = RECORD
            lat: INTEGER
            long: INTEGER
       END ;

  PROCEDURE ( loc : LOC ) What* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "gps coordinate"
  END What;

END GPS.  
```
Separately compiled code should be able to assign an interface value to behaviorally compatible types and call the appropriate methods in a type safe manner:

```
MODULE M;
IMPORT T, GPS, W;
VAR i: T.I; r: T.R; l:GPS.LOC; 
      x: ARRAY 32 OF CHAR;
      x: W.Identifier; 
BEGIN
      NEW(i); NEW(r); NEW(l);

      s := i;  s.What( x );  
      s := r;  s.What( x );  
      s := l;  s.What( x );  
END M.

```


