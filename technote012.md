# Tech Note 012 - Shift-Enter for Command Invocation
### Invoking commands in text using shift-enter

In RISC5 Oberon a mouse must be used to invoke a command. The Enter (or Return) key is consistently used in the text user interface for text editing, not for command invocation.

In Input.Mod the Enter key press is mapped to byte value 0AX and the Shift-Enter key press is mapped to 0DX. The Control-C key press is mapped to 03X.

Hmmm.... Control-C is already mapped for copy. Using it for interrupt as well isn't helpful.

In Oberon.Mod two new constants can be introduced (e.g. SHIFTENTER = 0DX; CONTROLC = 03X;) and the Loop procedure can be modified to handle occurances of these keypresses to a) invoke a command at the beginning of the line on which the cursor resides, ~and b) indicate that the user requests an abort of a user-initiated command.~



