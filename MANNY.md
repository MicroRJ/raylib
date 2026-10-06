# Building Raylib with Manny

This branch is a focused case study for [Manny](https://github.com/MicroRJ/Manny),
not a replacement for Raylib's complete multi-platform build system.

The root [`build.elf`](build.elf) describes one concrete configuration:

- Windows x64;
- a static release library;
- OpenGL 3.3;
- `clang-cl` with the MSVC linker and librarian;
- 121 Windows-compatible examples.

The branch is based on upstream Raylib commit
`c064eefe26ce1aaf54e665997ec746fa673e237c`. The pthread-based
`core_loading_thread` example is excluded. Runtime resources are not staged, so
examples that load `resources/...` must be launched with the appropriate working
directory.

## Build it

Install [Manny](https://github.com/MicroRJ/Manny) and Visual Studio's Desktop
development with C++ workload, including `clang-cl`. Then run this from the
repository root:

```powershell
manny --cache-vcvars
manny build.elf build --workers 8
```

The first command initializes Manny's cached Visual Studio environment and only
needs to be run once.

To build only the static library:

```powershell
manny build.elf library --workers 8
```

The full build produces:

- `build/raylib.lib`;
- 121 executables under `build/examples`;
- 129 object files under `build/obj`.

## What was verified

The case study was reproduced from a clean clone on Windows with Manny
`0.3.0-dev`, elf `0.2.0-dev`, and `clang-cl` 19.1.5.

| Check | Result |
| --- | --- |
| Clean build | 251 tasks completed; 121 examples linked |
| No-op build | All 251 tasks up to date; 101 ms on the test machine |
| One example source changed | That example's compile and link tasks rebuilt |
| `raylib.h` changed | 126 dependent compilations, the archive, and 121 links rebuilt |
| Compiler command changed | All 251 tasks rebuilt |

Two clean verification builds took 17.0 and 23.7 seconds on the test machine.
These timings describe one machine and are not presented as a benchmark against
Raylib's existing build systems.

## What this demonstrates

Manny's unchanged core was sufficient to model, parallelize, and incrementally
build this nontrivial Windows slice. The project-specific description is 153
lines of elf and lowers directly into Manny tasks.

Raylib's existing build definitions support substantially more: other operating
systems, architectures, graphics backends, shared libraries, installation,
packaging, Web and mobile targets, and additional toolchain configuration. This
case study does not claim equivalent coverage.
