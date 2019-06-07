# Tech Note 012 - Shift-Enter for Command Invocation
### Invoking commands in text using shift-enter

In RISC5 Oberon a mouse must be used to invoke a command. The Enter (or Return) key is consistently used in the text user interface for text editing but not for command invocation.

In Input.Mod the Enter key press is mapped to byte value 0AX which is the ASCII value for Linefeed (A.K.A Newline) and the Shift-Enter key press is mapped to 0DX, which is the ASCII value for Carriage Return.

The Handle procedure in TextFrames dispatches key events to the Write procedure, including Newlines and Carriage Returns. The Handle procedure also dispatches mouse events to the Edit procedure, including middle-button-clicks for command invocation.

The Write procedure can be augmented to perform similarly to the (*MM: Call*) logic in the Edit procedure when receiving a Carriage Return.


