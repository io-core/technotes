# Tech Note 013- A Call Message and Boot Scripts
### Introducing TextFrames.CallMsg to enable boot scripts in Oberon

A defining characteristic of Oberon is how a user may command-click on any displayed text and if that text is a recognized command, that text will be executed by the system.

Oberon users often accumulate lists of commands to perform common and repeated functions in .Tool files, which are simple Oberon texts like any other text file.

Andreas Pirklbauer has introduced batch processing into Extended Oberon, allowing a user to cause a number of commands to be initiated in sequence with one click like so:

```
Oberon.Batch
   ORP.Compile A.Mod B.Mod C.Mod ~
   System.Watch ~
   MyModule.MyCommand A B C ~
   ~
```
Andreas' system also introduces return codes for determining if a command has completed successfully. Andreas' system modifies both the Outer and Inner Core of Oberon.

Michael Schierl has also introduced a batch processing system but his requires no changes to the Outer or Inner core. The syntax is slightly different:

```
Batch.Run
|> ORP.Compile A.Mod B.Mod C.Mod ~
|> System.Watch ~
|> MyModule.MyCommand A B C ~
||
```
Michael's system, rather than introducing return codes, provides a way to verify the output that a previous command has written to the System Log.

A mouse (or the keyboard, if technote 12 is followed) is used to invoke the Batch command. When this input is received the TextFrames module locates the command and executes the procedure. The Oberon.Batch or Batch.Run procuedure then takes over the reading of the text and the calling of commands.

It would be useful to have a "batch" text file execute on startup, periodically according to a schedule, or at other times automatically when conditions are met. 

Two small changes to TextFrames.Mod and one small change to System.Mod, with either of the above Batch systems, introduces automatic execution of "batch scripts" in Oberon.

This system adds a "Call" message to the set of messages understood by TextFrames in TextFrames.Mod:

```
     CallMsg* = RECORD (Display.FrameMsg)
       offset*: INTEGER
     END;
```

When the message is received by the Handle procedure in TextFrames.Mod, a Call is made with the offset provided by the message just as if the user had clicked on the text:

```
       UpdateMsg: IF F.text = M.text THEN Update(F, M) END | 
       CallMsg: Call(F,M.offset,FALSE)
```

In System Mod we need two new top level VARs:

```
    jobV: Viewers.Viewer;
    jobM: TextFrames.CallMsg;
```

In OpenViewers we need to allocate and open a Startup.Job viewer:

```
    Oberon.AllocateSystemViewer(0, X, Y);
    menu := TextFrames.NewMenu("Startup.Job", StandardMenu);
    main := TextFrames.NewText(TextFrames.Text("Startup.Job"), 0);
    jobV := MenuViewers.New(menu, main, TextFrames.menuH, X, Y)
```

And finally, as the last action in System initialization we cause the job to start:

```
  jobM.offset := 0;
  jobV.dsc.next.handle(jobV.dsc.next,jobM);
```

