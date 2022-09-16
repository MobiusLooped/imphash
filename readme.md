### Changes in this fork

* The original imphash.go file uses string concatenation that, when automated, creates massive memory allocation issues. These have been replaced with builder.String() to streamline and pare down memory required. When running against large batches, memory use generally hovers between 600MB - 1GB; original memory use regularly crashed the parser on systems with 80GB of RAM.
* Due to an issue with how memory is allocated, a bug was discovered when attempting to parse ELF files that had been malformed. There is now a check against the actual section sizes of the file (as opposed to trusting the size listed in the file headers).

### ImpHash for Go

* The imports are sorted by the library and function name, so re-ordering the imports doesn't change the import hash. However, that means the imports aren't the same as the `pefile` Python module, or other sources, such as VirusTotal.
* Fuzzy import hashes are achieved by using SSDeep to generate a fuzzy hash of the import information, after sorting. This was inspired by the [impfuzzy](https://github.com/JPCERTCC/impfuzzy) project.
* Additional features over other implementations:
  * ELF support: since ELFs also have imports, so why not? Don't expect the same hashes between file types.
  * Mach-O support: since Macho-O files have imports as well. However, the library and imported functions aren't paired, so they're not hashed together, but instead all in one sorted list. Includes Fat Mach-O files as well.
* Trying to run ImpHash on a static file will result in an error, which is expected since there's no import data to crunch in that case.
* Since Go's native executable format parser is used, some weird files, like malware, may not be parseable and will result in an error.
* The main goal of this project is to provide an additional file similarity metric to [MalwareDB](https://github.com/malwaredb), but can be used with other projects as well.

#### Data Structure
The function `ImpHashFromBytes()` returns the below structure, and possibly an `error` object.

    type ImpHashResult struct {
	    ImpHash string   // MD5 hash of the sorted imports
	    ImpFuzzy string  // SSDeep hash of the sorted imports
	    ImpString string // The sorted string which was hashed.
    }
