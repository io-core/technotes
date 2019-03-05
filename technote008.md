# Tech Note 008 - Packages
### Package source code and artifacts with requirements and semantic versioning

* A collection of related modules may be packaged together for a unified release
* Modules may depend on minimum and maximum versions of other modules
* Tooling may assist in obtaining packages (todo)
* Compatibility across distributions of Oberon should be supported
* Deterministic builds should be possible (todo)
* Signatures should be provided and checkable (todo)

Example package file `System.Pkg` which, when its requirements are also fetched, encompases the original Project Oberon files:

```
package [core]/System v5.1.0

requires (
        [core]/Kernel v5.0.0
        [core]/Files v5.1.0
        [core]/Modules v5.1.0
        [core]/Oberon v5.1.0
        [core]/Edit v5.1.0
        [core]/Draw v5.1.0
        [core]/Paint v1.0.0
)

provides (
	Blink.Mod
	RISC.Mod
	Tools.Mod
	BootLoad.Mod
	Net.Mod
	System.Mod        
	PCLink1.Mod
	RS232.Mod
	Math.Mod
	PIO.Mod
	SCC.Mod
	System.Tool
)

```

An Oberon application package may be compatible with both the io-core system and with the original Project Oberon 2013 system, as below:

```
package [extra]/Score v0.1.0

requires (
        [core]/System v5.0.0
)

provides (
        Notes.Mod
        Score.Mod
        TabFrames.Mod
        Tabs.Mod
)

```


