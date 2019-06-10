# Tech Note 012 - Shift-Enter for Command Invocation
### Invoking commands in text using shift-enter

In RISC5 Oberon a mouse must be used to invoke a command. The Enter (or Return) key is consistently used in the text user interface for text editing but not for command invocation.

In `Input.Mod` the Enter key press is mapped to byte value 0AX which is the ASCII value for Linefeed (A.K.A Newline) and the Shift-Enter key press is mapped to 0DX, which is the ASCII value for Carriage Return.

The Handle procedure in `TextFrames.Mod` dispatches key events to the Write procedure, including Newlines and Carriage Returns. The `Handle` procedure also dispatches mouse events to the `Edit` procedure, including middle-button-clicks for command invocation.

The `Write` procedure can be augmented to perform similarly to the (*MM: Call*) logic in the `Edit` procedure when receiving a Carriage Return, as follows (for Unicode capable Oberon; for regular Oberon use `ch` instead of `codepoint`, etc:)

```
(* new VARs *)
      loc: Location; keysum: SET;
      patadr, bpos, pos, lim: LONGINT;
      bx, ex, ox, dx, u, v, w, h: INTEGER;

(* after the check for ctrl-x, cut *)
    ELSIF codepoint = ORD(CR) THEN (*Shift-Enter a.k.a. CR*)
      IF F.trailer.next # F.trailer THEN
        LocateLine(F, F.carloc.y - F.Y, loc);
        lim := loc.org + loc.lin.len - 1;
        bpos := loc.org; bx := F.left; 
        pos := loc.org; ox := F.left;
        Texts.OpenReader(R, F.text, loc.org); Texts.ReadUnicode(R, nextCodepoint);
        WHILE (pos # lim) & (nextCodepoint <= ORD(" ")) DO (*scan gap*)
          Fonts.GetUniPat(R.fnt, nextCodepoint, dx, u, v, w, h, patadr);
          INC(pos, Texts.UnicodeWidth(nextCodepoint)); ox := ox + dx; Texts.ReadUnicode(R, nextCodepoint)
        END;
      ELSE pos := 0  (*<----*)
      END;
      IF (pos >= 0) THEN Call(F, pos, 2 IN keysum) END

```

The above code instructs the Oberon system to call the *first* command on the same line that the caret is positioned on, when shift-enter is pressed.
