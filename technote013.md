# Tech Note 013- A Call Message and Boot Scripts
### Introducing TextFrames.CallMsg to enable boot scripts in Oberon

A defining characteristic of Oberon is how a user may command-click on any displayed text and if that text is a recognized command, that text will be executed by the system.

Oberon users often accumulate lists of commands to perform common and repeated functions in .Tool files, which are simple Oberon texts like any other text file.

Andreas Pirklbauer has introduced batch processing into [Extended Oberon](https://github.com/andreaspirklbauer/Oberon-extended), allowing a user to cause a number of commands to be initiated in sequence with one click like so:

```
Oberon.Batch
   ORP.Compile A.Mod B.Mod C.Mod ~
   System.Watch ~
   MyModule.MyCommand A B C ~
   ~
```
Andreas' system also introduces return codes for determining if a command has completed successfully. Andreas' system modifies both the Outer and Inner Core of Oberon.

Michael Schierl has also introduced a [Batch](https://github.com/schierlm/OberonEmulator/blob/master/Oberon/Batch.Mod.txt) processing system but his requires no changes to the Outer or Inner core. The syntax is slightly different:

```
Batch.Run
|> ORP.Compile A.Mod B.Mod C.Mod ~
|> System.Watch ~
|> MyModule.MyCommand A B C ~
||
```
Michael's system, rather than introducing return codes, provides a way to verify the output written to the System Log by a command.

A mouse (or the keyboard, if [technote 12](https://github.com/io-core/technotes/blob/main/technote012.md) is followed) is used to invoke the Batch command. When this input is received the TextFrames module locates the command and executes the procedure. The Oberon.Batch or Batch.Run procuedure then takes over the reading of the text and the calling of commands.

It would be useful to have a "batch" text file execute on startup, periodically according to a schedule, or at other times automatically when conditions are met. 

Two small changes to `TextFrames.Mod` and three small changes to `System.Mod` with either of the above Batch systems will introduce automatic execution of "batch scripts" in Oberon.

The changes to `TextFrames.Mod` change the symbol file and therefore require a recompilation of all modules that rely on TextFrames (e.g. the Outer Core.)

This system adds a "Call" message to the set of messages understood by TextFrames in `TextFrames.Mod`:

(* Between the definitions of UpdateMsg and CopyOverMsg *)
```
     CallMsg* = RECORD (Display.FrameMsg)
       offset*: INTEGER
     END;
```

When the message is received by the Handle procedure in `TextFrames.Mod`, a Call is made with the offset provided by the message just as if the user had clicked on the text:

(* Replacing the UpdateMsg line in Handle *)
```
       UpdateMsg: IF F.text = M.text THEN Update(F, M) END | 
       CallMsg: Call(F,M.offset,FALSE)
```

In `System.Mod` we need two new top level VARs:

(* At the end of the VAR section at the top of the file *)
```
    jobV: Viewers.Viewer;
    jobM: TextFrames.CallMsg;
```

In the `OpenViewers` procedure of `System.Mod` we need to allocate and open a Startup.Job viewer:

(* Just before END OpenViewers; *)
```
    Oberon.AllocateSystemViewer(0, X, Y);
    menu := TextFrames.NewMenu("Startup.Job", StandardMenu);
    main := TextFrames.NewText(TextFrames.Text("Startup.Job"), 0);
    jobV := MenuViewers.New(menu, main, TextFrames.menuH, X, Y)
```

As the last action in `System.Mod` initialization we cause the job to start:

(* At the end of the file, just before END System. *)
```
  jobM.offset := 0;
  jobV.dsc.next.handle(jobV.dsc.next,jobM);
```

Finally, a file named Startup.Job should be placed in the file system, for example:

```
Batch.Run
|> ORP.Compile Kernel.Mod/s FileDir.Mod/s Files.Mod/s Modules.Mod/s~
|> Batch.Collect
|> ORP.Compile Fonts.Mod/s Texts.Mod/s Input.Mod/s Display.Mod/s ~
|> Batch.Collect
|> ORP.Compile Viewers.Mod/s Oberon.Mod/s ~
|> Batch.Collect
|> ORP.Compile MenuViewers.Mod/s Graphics.Mod/s GraphicFrames.Mod/s ~
|> Batch.Collect
|> ORP.Compile TextFrames.Mod/s Batch.Mod/s System.Mod/s ~
|> Batch.Collect
|> ORP.Compile ORS.Mod/s ORB.Mod/s ORG.Mod/s ORP.Mod/s ~
|> Batch.Collect
|> ORP.Compile Edit.Mod/s Tools.Mod/s ORTool.Mod/s ~
|> Batch.Collect
|>ORP.Compile Draw.Mod/s ~
||
```

The above Startup.Job script expects the `Batch.Run` command to be avaliable and the Batch module imports `FileDir.Mod` which may not have a symbol file in a stripped version of the Oberon Disk image, so before compiling the changed `TextFrames.Mod` and `System.Mod`:

```
ORP.Compile FileDir.Mod Batch.Mod ~
```

After the above command is executed the Startup.Job script can be run manually (middle-clicking on `Batch.Run`) to regenerate the rest of the Outer Core of Oberon.

After the Outer Core is regenerated the system can be rebooted and the Startup.Job script should run automatically on system startup. The startup script may be modified to perform any other commands desired just by editing and saving the changes.

![Startup.Job](https://github.com/io-core/Technotes/blob/main/images/StartupJob.png "Startup.Job")

