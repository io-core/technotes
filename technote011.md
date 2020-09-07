# Tech Note 011 - Scripting
### Automating command application with scripts

It is convenient to automate the application of commands. [LIL](http://runtimeterror.com/tech/lil) shows one way of implementing a scripting language that may interface well with Oberon's text user interface.

* lines of "Module.Command argument" should be supported for scripting familiar operations
* Conditionals, variable substituion, etc. are desireable
* Pipes would be nice
* prompt interaction is useful
* environment variables add customizability
* Return values and error codes are needed 

In A2 Oberon the [Oberon.Configuration](https://en.wikibooks.org/wiki/Oberon/A2/Oberon.Configuration.Mod) module provides an example of batch execution of Oberon commands

Michael Schierl's [Batch.Mod](https://github.com/schierlm/OberonEmulator/blob/a57f61ecca3aeb241004b6d5c48261ce20d5f03d/Oberon/Batch.Mod.txt) also provides a facility to verify the Log buffer against a checksum and testing against the result for script success or failure.



