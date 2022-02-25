# Known limitations
Most Go apps can be compiled and run with EGo without modifications. However, there are some limitations:

* An EGo app is a single process. Spawning other processes isn't possible.
* cgo support is experimental
  * Libraries must be statically linked. Shared objects are unsupported.
* Stripped executables (e.g., `ego-go build -ldflags -s`) are unsupported

## (Partially) unsupported packages
These usually compile, but return an error at runtime.

* `os`: `Pipe` and `StartProcess` are unsupported
* `os/exec`: spawning processes is unsupported

## cgo: Unsupported libc functions
Using these functions causes a build, sign, or runtime error.

* `exec` family
* `fork`
* `pipe`
* `posix_spawn`
* `mmap`: partially supported; memory-mapped files are unsupported
