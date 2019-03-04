# Tech Note 008 - Packages
### Package source code and artifacts with requirements and semantic versioning

* A collection of related modules may be packaged together for a unified release
* Modules may depend on minimum and maximum versions of other modules
* Tooling may assist in obtaining packages (todo)
* Deterministic builds should be possible (todo)
* Signatures should be provided and checkable (todo)

Example package file `Edit.Pkg`:

```
package [io-core]/Edit v5.1.0

requires (
	[io-core]/Files v5.1.0
        [io-core]/Modules v5.1.0
        [io-core]/Oberon v5.1.0
)

provides (
	Edit.Mod
	Fonts.Mod
	TextFrames.Mod
	Texts.Mod
)
```


