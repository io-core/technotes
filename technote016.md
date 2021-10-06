# Tech Note 016 - Command parameters, context, argc/argv, the clipboard, messages, and namespaces in IO
### Provide a unified namespace for multi-language programming and user interaction in IO

* At command invocation time, whether on click or in a command script or on receipt of a message:
 1 Available modules (loaded or not) provide a namespace for commands in the text user interface and command scripts and message sends
 2 The modules namespace is a linear concatenation of Module.Command(s)
 3 bin paths or union mounts per context can tailor the modules visible in the namespace (linear per context)
 4 independent instances of modules on import can tailor the behavior of the visible modules per importing module
 5 There may be relevent UI state (selected text, marker positions, clipboard stack of content)
 6 There may be relevent environment values (per context)
 7 There may be open streams for input and output and error
* At compile time 
 1 


* something
 1 saving a referenced global variable on the stack upon procedure entry
 2 accessing and modifying the global variable as usual
 3 restoring the cached value upon procedure exit
* A `DYN` keyword and compiler assistance can reduce boilerplate
* DYN variables may enable the common lisp condition system in Oberon
* This kind of dynamic scope may only work local to the thread of execution
* This kind of dynamic scope is ineficient for large objects

