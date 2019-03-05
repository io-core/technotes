# Tech Note 008 - Packages
### Packaging in Oberon with requirements and semantic versioning

It would be useful to have a package management system for Oberon code and artifacts that assists the programmer in handling dependencies and versions. It would be helpful if such a system were not tied to a particular distribution of Oberon but would allow a package (if conservatively coded) to be automatically obtained and integrated across implementations of the system.

Packaging goals:

* A collection of related modules may be packaged together for a unified release
* Modules may depend on minimum and maximum versions of other modules
* Tooling to assist in obtaining packages
* Compatibility across distributions
* Deterministic builds
* Signatures and verification

### Original Oberon

The original Project Oberon can be viewed as one [package of modules and resources](https://github.com/io-orig/System), version 5 (as compared to Versions 4 or 3 or Ceres or Lilith Oberon), which does not require any other packages:

```
package [core]/System v5.0.20190404

requires ()
provides (
	Blink.Mod     GraphicFrames.Mod  Oberon.Mod      RS232.Mod
	BootLoad.Mod  Graphics.Mod       ORB.Mod         SCC.Mod
	Checkers.Mod  GraphTool.Mod      ORC.Mod         Sierpinski.Mod
	Curves.Mod    Hilbert.Mod        ORG.Mod         SmallPrograms.Mod
	Display.Mod   Input.Mod          ORP.Mod         Stars.Mod
	Draw.Mod      Kernel.Mod         ORS.Mod         System.Mod
	EBNF.Mod      MacroTool.Mod      ORTool.Mod      TextFrames.Mod
	Edit.Mod      Math.Mod           PCLink1.Mod     Texts.Mod
	FileDir.Mod   MenuViewers.Mod    PIO.Mod         Tools.Mod
	Files.Mod     Modules.Mod        Rectangles.Mod  Viewers.Mod
	Fonts.Mod     Net.Mod            RISC.Mod
)
 ```
### Dividing Original Oberon

Oberon can be divided into logically grouped modules by functionality and dependencies. IO breaks the system down into the following packages:

 Package | Description
 --------|------------
Kernel   | With only Kernel.Mod, this package does not rely on any other packages.
Files    | With Files.Mod and FileDir.Mod, this package depends on Kernel and encapsulates Oberon's storage subsystem.
Modules  | Requiring the above two packages, this contains Modules.Mod and completes the Inner Core of Oberon.
Oberon   | Providing Input.Mod, Display.Mod, Viewers.Mod, MenuViewers.Mod and Oberon.Mod, this package has a mutual dependency with Edit for text handling.
Edit     | Providing Fonts.Mod, TextFrames.Mod, Texts.Mod and Edit.Mod, this package has a mutual depencency with Oberon.
Build    | Providing the Oberon compiler and tools, Build package depends on the Oberon and Text packages.
Draw     | Providing the line graphics and editing subsystem, this package depends directoy on Edit, Oberon, Modules, and Files.
System   | With the previous packages, this rounds out the classic Oberon user experience.

The package file `System.Pkg` which, when its requirements are also fetched, encompases the original Project Oberon files. The minor number has been bumped from 0 to 1 for some required packages because of changes to support Unicode fonts, other screen geometries, etc. but the required Kernel remains unchanged from Original Oberon in this example. 

### Pkg File Format

The Pkg file gives a package name and version number, and lists the other packages that this package directly relies on. Those packages may require other packages as well. The package also lists the modules and other resources provided by the package. Using the System package as an example:

```
package [core]/System v5.1.0

requires (
        [core]/Kernel v5.0.0
        [core]/Files v5.1.0
        [core]/Modules v5.1.0
        [core]/Oberon v5.1.0
        [core]/Edit v5.1.0
        [core]/Draw v5.1.0
        [core]/Build v5.1.0
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

### Compatibility between Distributions

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
### Get Command

Oberon needs a tool to get packages. While V5 Oberon does not have a TCP/IP stack, an external tool can perform package management. The Get tool may be used as follows:

* Get latest [core]/Paint
* Get specific [io-extra]/Score v1.2.3
* Get backto [io-core]/Kernel v5.0.20190404
* Get latest github.com/privaterepo/Supercool
* Get latest everything
* Get changelist
* Get versions

### Get Environment Settings

* List of package repositories
* [core] [extra] etc. paths for default distribution sources
* Line Ending conversion (LF <--> CRLF) for distributions keeping Original Project Oberon text format.
* Single directory / separate directories local file deposition

