# Tech Note 008 - Packages

# (*incomplete*)

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
k,v1,v2
package,System,v5.0.20160620
from,github.com/io-orig,
license,PROJECTOBERON,NW PR JG
doc,core,all-in-oine collection of Oberon source.
p,System.Mod,215e86c25a1499d0ee5e63f1a1f3ce4230422f12964752155ea291b478773a08
p,MenuViewers.Mod,c91c52dd1441f545057467b6bc3122fa0a09550a0f8d7dc40e812c43f3602b7b
p,PIO.Mod,3a2b9d960dae5ddbbdd1ee6db7667b47670d80957eca866264940d0cd704c0cf
p,Tools.Mod,309c689adf4a0148e6f373bac1dfe7a2747d8b5d1492ea78507b933dd14816aa
p,PCLink1.Mod,b49cd9b51c89a3adc27b5f317105d8bb9c4f4addb1bef2921ade6b432c6f2ad3
p,RS232.Mod,76e277553b26612b90160f41b8341dfc1f7648d8ac89b78a48b53a91a989630f
p,Net.Mod,8b653ac5aa471c6fcd6637109c8f3ae2f88b127ce3051c5d77c3496a34c690bd
p,SCC.Mod,a172c720b47b5508b4fb27381dc9665de99c7470f6b9c9397ece5ff08cf17725
p,Batch.Mod,309c689adf4a0148e6f373bac1dfe7a2747d8b5d1492ea78507b933dd14816aa
p,Halt.Mod,e187550d5b645d61fc2e0a80ab98e6030e809724927385142c19f5dad4a11650
p,ORP.Mod,16a4e0185784fcb508367a89e6c965d6fe72673abd2038b174bbe90d65aab168
p,ORG.Mod,cc84a2c0a7c9fa7244a02dbdc7e7f7bd15cfeb7d3903d14950666c95626897e0
p,ORB.Mod,5f8cc22f2e08a93fde241085fb76d187b84bbf9d2ef30aee9a64fd158a0a635b
p,ORS.Mod,d7095213ac8cec4fe11bcb5400dd8ee120218d31c58d9012cbf74e6a7d573161
p,Draw.Mod,7009585c52f0bd07a67a5e3d9dbfe88a0079d07b630cba2723eed72bb9c70a24
p,Graphics.Mod,22cf2e57b1576966593a68d20f5cecdd6a3c43282658e1e8fb5f8af22d5246c4
p,MacroTool.Mod,5f158470102b0e5a8a38659afe4b17b353fd46a07b457684a0e8f48f7fc463ef
p,GraphicFrames.Mod,dfaa68c2d5a3313990ea10d055315b0c9a26cb987e72778026e6cba1b04bc92d
p,GraphTool.Mod,cc5ebb6cec2845ef5f7066d052c22523ff92337d898eb8c5698d5bc4153fcd44
p,Rectangles.Mod,e187550d5b645d61fc2e0a80ab98e6030e809724927385142c19f5dad4a11659
p,Curves.Mod,4efa3b61094883b1ae9e2563bc4dbf8c56e59216fb6abb59b9293659d096bd29
p,Edit.Mod,0eadd23ec2b457f4b427398f6a3b77044ffcf34f1c077b7e9912600749dcfc0e
p,Fonts.Mod,905340fe91261b4a292f5528846114099f8f25250b2cae64e9d5cd39736aa2de
p,TextFrames.Mod,8de2e6b4461360cd3f0605a0a9c5057fbce77739593894a1aaf8ff625ec1e845
p,Texts.Mod,16a4e0185784fcb508367a89e6c965d6fe72673abd2038b174bbe90d65aab168
p,Blink.Mod,5f8cc22f2e08a93fde241085fb76d187b84bbf9d2ef30aee9a64fd158a0a635b
p,Math.Mod,aa1a5c2441053298408b0d2844e29470274860cb53b187b7a96eea8df262f014
p,RISC.Mod,f4c2902238b46708cf2de5d80b148b74e75825c04994b02b04d6451f500d8e91
p,Sierpinski.Mod,aa1a5c2441053298408b0d2844e29470274860cb53b187b7a96eea8df262f014
p,Hilbert.Mod,cc84a2c0a7c9fa7244a02dbdc7e7f7bd15cfeb7d3903d14950666c95626897e0
p,Checkers.Mod,d7095213ac8cec4fe11bcb5400dd8ee120218d31c58d9012cbf74e6a7d573161
p,FileDir.Mod,df0cb5feb860c37370b57009c21905a25886c057e02f266a0ebc94465e82f9ab
p,Files.Mod,af20a42cac9e11b5b9d8e6036ca891ef6b3d7eb08747f027f52c2cb450fbf9b4
p,Modules.Mod,83d0bbda06675ac366d3b2f61a6511e398c0ef65646059120c89796c1919ced6
p,Oberon.Mod,cc84a2c0a7c9fa7244a02dbdc7e7f7bd15cfeb7d3903d14950666c95626897e0
p,Viewers.Mod,d7991eabdff1cb6b06c4a085f1a5a05bf10cc03398bcf03b9d47241865d357ae
p,Display.Mod,d7095213ac8cec4fe11bcb5400dd8ee120218d31c58d9012cbf74e6a7d573161
p,Input.Mod,4eb36a06d406512335c595b830600cf9df42eb6e351c22d7ca27358c7ec5ce5d

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
k,v1,v2
package,System,v5.0.20160620
from,github.com/io-core,
license,PROJECTOBERON,NW PR JG
doc,core,prepares the user interface and manages viewers for the user.
r,[core]/Kernel,v5.0.0
r,[core]/Files,v5.0.0
r,[core]/Modules,v5.0.0
r,[core]/Oberon,v5.0.0
r,[core]/Edit,v5.0.0
r,[core]/Draw,v5.0.0
p,System.Mod,215e86c25a1499d0ee5e63f1a1f3ce4230422f12964752155ea291b478773a08
p,MenuViewers.Mod,c91c52dd1441f545057467b6bc3122fa0a09550a0f8d7dc40e812c43f3602b7b
p,PIO.Mod,b49cd9b51c89a3adc27b5f317105d8bb9c4f4addb1bef2921ade6b432c6f2ad3
p,Tools.Mod,309c689adf4a0148e6f373bac1dfe7a2747d8b5d1492ea78507b933dd14816aa
p,PCLink1.Mod,b49cd9b51c89a3adc27b5f317105d8bb9c4f4addb1bef2921ade6b432c6f2ad3
p,RS232.Mod,76e277553b26612b90160f41b8341dfc1f7648d8ac89b78a48b53a91a989630f
p,Net.Mod,8b653ac5aa471c6fcd6637109c8f3ae2f88b127ce3051c5d77c3496a34c690bd
p,SCC.Mod,a172c720b47b5508b4fb27381dc9665de99c7470f6b9c9397ece5ff08cf17725
p,Batch.Mod,c91c52dd1441f545057467b6bc3122fa0a09550a0f8d7dc40e812c43f3602b7b
p,Halt.Mod,b49cd9b51c89a3adc27b5f317105d8bb9c4f4addb1bef2921ade6b432c6f2ad3
p,Hilbert.Mod,215e86c25a1499d0ee5e63f1a1f3ce4230422f12964752155ea291b478773a08
p,Sierpinski.Mod,b49cd9b51c89a3adc27b5f317105d8bb9c4f4addb1bef2921ade6b432c6f2ad3
p,Stars.Mod,309c689adf4a0148e6f373bac1dfe7a2747d8b5d1492ea78507b933dd14816aa
p,Checkers.Mod,b49cd9b51c89a3adc27b5f317105d8bb9c4f4addb1bef2921ade6b432c6f2ad3
p,Clipboard.Mod,b49cd9b51c89a3adc27b5f317105d8bb9c4f4addb1bef2921ade6b432c6f2ad3


```

### Compatibility between Distributions

An Oberon application package may be compatible with both the io-core system and with the original Project Oberon 2013 system, as below:

```
(* provide example *)

```
### Package Command

Oberon needs a tool to get packages. While V5 Oberon does not have a TCP/IP stack, an external tool can perform package management. The Package tool may be used as follows:

* Package latest [core]/Paint
* Package specific [io-extra]/Score v1.2.3
* Package backto [io-core]/Kernel v5.0.20190404
* Package latest github.com/privaterepo/Supercool
* Package latest everything
* Package changelist
* Package versions

### Package Environment Settings

* Packages.csv and Packaging.csv
* List of package repositories
* [core] [extra] etc. paths for default distribution sources
* Line Ending conversion (LF <--> CRLF) for distributions keeping Original Project Oberon text format.
* Single directory / separate directories local file deposition


### Oberon Package Index

* It would be nice to have one

### [core] annd [extra] as cross-distribution labels

A dependency on a [core]/System package or [Extra]/Other package should be portable across distributions of the Oberon system

