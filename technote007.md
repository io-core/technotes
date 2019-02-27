# Tech Note 007 - Cache by Hash
### Use a hash cache to find already loaded resources

* Identical Fonts, images, sound files, etc. may be used in various different contexts which may not coordinate
* A hash of the identifier (e.g. filename) uses a fixed amount of space and can be stored in a b-tree 
* Need something like a weak pointer
* File system integrated
* Cacheing records (structured content) *not* hidden from GC -- enabling automatic reclamation

