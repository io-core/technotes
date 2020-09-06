# Tech Note 014 - Interfaces for Oberon-2
## Rationale

Oberon-2 extends the Oberon programming language to support type-bound procedures. Each record with associated type-bound procedures (a.k.a. its 'methods') has a pointer to a table that is used by calling code to locate the entry points of the methods. Typical object oriented programming in Oberon-2 leverages this arrangement.

Type safety in Oberon-2 requires pointers to only point to a specific base type or an extension of that base type of record. A pointer may not have any other type of record assigned to it even if the two base types are otherwise structurally or behaviorally compatible. 

It is sometimes useful to be able to refer to items that are implemented differently but which exhibit a compatible subset of behavior (i.e. have some methods with matching type signatures) but type safety should not be sacrificed for expediency. The 'go' programming language implements type-safe behavior matching via 'Interfaces' while the 'rust' programming language calls this 'dynamic traits.'

Some relatively small modifications to the Oberon-2 compiler and Oberon system enable Interfaces in Oberon.

## Syntax

To express Interfaces in an Oberon-2 program the syntax may be extended with a new keyword 'INTERFACE' and new syntax for declaring an Interface type, for example: 

```
MODULE W;
  TYPE
       Describer* = INTERFACE OF
            PROCEDURE What* (VAR a: ARRAY OF CHAR) ;
            PROCEDURE String* (VAR a: ARRAY OF CHAR) ;
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

  PROCEDURE ( i : I ) String* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "1"
  END String;

  PROCEDURE ( r : R ) What* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "float"
  END What;
  
  PROCEDURE ( r : R ) String* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "3.14"
  END String;
  
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

  PROCEDURE ( loc : LOC ) String* (VAR a: ARRAY OF CHAR) ;
  BEGIN a := "32.7157 N, 117.1611 W"
  END String;

END GPS.  
```
Separately compiled code should be able to assign an interface value to behaviorally compatible types and call the appropriate methods in a type safe manner:

```
MODULE M;
IMPORT T, GPS, W;
VAR i: T.I; r: T.R; l:GPS.LOC; 
      x: ARRAY 32 OF CHAR;
      d: W.Describer; 
BEGIN
      NEW(i); NEW(r); NEW(l);

      d := i;  d.What( x ); d.String( x );  
      d := r;  d.What( x ); d.String( x );
      d := l;  d.What( x ); d.String( x );
END M.

```

The computational cost for this abstraction is paid in the assignment of a record pointer to an interface value. This is when the runtime must prepare an appropriate method table for this interface as applied to this record type (an interface method table cache may help) and raise an error if the type signatures of the methods do not match. This cost can be minimized and is not paid if interfaces are not used.


## Implementation

Introducing the above functionality in the version of Oberon-2 as implemented in Andreas Pirklbauer's Oberon-Extended (2020) implementation starts with additions to ORB.Mod to introduce a new form value:

```
    (* form values*)
      Byte* = 1; Bool* = 2; Char* = 3; Int* = 4; Real* = 5; Set* = 6;
      Pointer* = 7; Interface* = 8; NilTyp* = 9; NoTyp* = 10; Proc* = 11;
      String* = 12; Array* = 13; Record* = 14; TProc* = 15;
      Ptrs* = {Pointer, Interface, NilTyp}; Procs* = {Proc, NoTyp};
```

Also in ORB.Mod add a check for NIL in UpdateLinks within NewMethod:

```
    BEGIN
     IF rec.typobj # NIL THEN
...
     END
    END UpdateLinks;
```

In ORS.Mod we introduce a new keyword:

```  
CONST IdLen* = 32;
    NKW = 35;  (*nof keywords*)
...
    array* = 60; record* = 61; pointer* = 62; interface* = 63; const* = 64; type* = 65;
    var* = 66; procedure* = 67; begin* = 68; import* = 69; module* = 70; eot = 71;
...
  EnterKW(procedure, "PROCEDURE");
  EnterKW(interface, "INTERFACE");
```
Making the above modifications changes every symbol file of every module so the while system must be re-built and a new inner-core installed before continuing.

With the above changes, ORP.Mod may be extended to parse INTERFACE type definitions (just before PROCEDURE Type0) like this:

```
  PROCEDURE InterfaceType(VAR type: ORB.Type; expo: BOOLEAN);
    VAR obj, objr, obj0, new, bot, base, proc, redef: ORB.Object;
      typ, tp, ftype: ORB.Type; id, procid, recid: ORS.Ident;
      offset, off, n, parblksize: LONGINT; expo0: BOOLEAN; size: LONGINT; nofpar: INTEGER;
  BEGIN
    Check(ORS.of, "no OF");
    NEW(type); type.base := NIL; type.mno := -level; type.nofpar := 0; type.len := 0; offset := 0; bot := NIL;
    type.form := ORB.Interface; type.dsc := bot; type.size := ORG.WordSize*2; type.typobj := NIL;

    WHILE sym = ORS.procedure DO
      ORS.Get(sym); Check(ORS.ident, "no identifier"); ORS.CopyId(procid);
      NEW(ftype); ftype.base := ORB.noType; ftype.size := ORG.WordSize; ftype.len := 0;  (*len used as heading of fixup chain of forward refs*)
      ORB.NewMethod(type, proc, redef, procid);
      ftype.form := ORB.TProc; proc.type := ftype; proc.val := -1;
      ORS.Get(sym); proc.expo := expo;
      IF expo THEN proc.exno := exno; INC(exno); 
        procid := "@"; ORB.NewObj(obj, procid, ORB.Const); obj.name[0] := 0X; (*dummy to preserve linear order of exno*)
        obj.type := proc.type; obj.dsc := proc; obj.exno := proc.exno; obj.expo := FALSE;
      END ;
      ORB.OpenScope; INC(level);  parblksize := 4;
      recid := "@"; ORB.NewObj(obj, recid, ORB.Var);  (*insert receiver as first parameter -- as an interface, will be replaced with target*)
      obj.type := type; obj.rdo := FALSE; obj.lev := level; obj.val := parblksize;
      INC(parblksize, ORG.WordSize);
      ProcedureType(ftype, parblksize); ftype.dsc := ORB.topScope.next; INC(ftype.nofpar);  (*formal parameter list*)
      ORB.CloseScope; DEC(level);
      proc.type:=ftype;         
      IF sym = ORS.semicolon THEN ORS.Get(sym) ELSIF sym # ORS.end THEN ORS.Mark(" ; or END") END
    END
  END InterfaceType;
```

In PROCEDURE Type0 add a check for ORS.interface between ORS.record and ORS.pointer:

```
    ELSIF sym = ORS.interface THEN ORS.Get(sym); InterfaceType(type, expo); Check(ORS.end, "no END");
```

in ORP StatSequence needs to know how to store an interface: 

```
            ELSIF (x.type.form = ORB.Interface) & (y.type.form = ORB.Pointer) THEN
              ORG.StoreInterface(x, y)
            ELSE ORS.Mark("illegal assignment")
```

In ORG.Mod introduce the StoreInterface procedure after StoreStruct:

```
  PROCEDURE StoreInterface*(VAR x, y: Item); (* x := y *) (* TODO: Build interface table *)
    VAR op: LONGINT;
  BEGIN  load(y);
    op := Str ;
    IF x.mode = ORB.Var THEN
      IF x.r > 0 THEN (*local*) Put2(op, y.r, SP, x.a + frame) ELSE PutPair(x.r, op, y.r, RH, x.a, 2) END
    ELSIF x.mode = ORB.Par THEN Put2(Ldr, RH, SP, x.a + frame); Put2(op, y.r, RH, x.b);
    ELSIF x.mode = RegI THEN Put2(op, y.r, x.r, x.a); DEC(RH);
    ELSE ORS.Mark("bad mode in Store")
    END ;
    DEC(RH)
  END StoreInterface;
```


In ORG.Mod the procedure StoreStruct needs a way to store to an Interface value:

```
    ELSIF x.type.form = ORB.Record THEN Put1a(Mov, RH, 0, x.type.size DIV 4)
    ELSIF x.type.form = ORB.Interface THEN Put1a(Mov, RH, 0, x.type.size DIV 4)
```
