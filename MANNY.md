# Building Raylib with Manny

This branch is a focused case study for [Manny](https://github.com/MicroRJ/Manny),
not a replacement for Raylib's complete multi-platform build system.

The root [`build.elf`](build.elf) describes one concrete configuration:

- Windows x64;
- a static release library;
- OpenGL 3.3;
- `clang-cl` with the MSVC linker and librarian;
- all 226 examples in the current tree.

The branch is based on upstream Raylib commit
`e18de0722427a5353c6044a0b7dc57ae07d8ef11`. Runtime resources are not staged,
so examples that load `resources/...` must be launched with the appropriate
working directory.

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

To remove the build outputs and Manny's incremental state:

```powershell
manny build.elf clean
```

The full build produces:

- `build/raylib.lib`;
- 226 executables under `build/examples`;
- 233 object files under `build/obj`.

## What was verified

The case study was reproduced from a clean clone on Windows with Manny
`0.3.0-dev`, elf `0.2.0-dev`, and `clang-cl` 19.1.5.

| Check | Result |
| --- | --- |
| Clean build | 460 tasks completed; 226 examples linked |
| No-op build | All 460 tasks up to date; 174-186 ms on the test machine |
| One example source changed | That example's compile and link tasks rebuilt |
| `raylib.h` changed | 231 dependent compilations, the archive, and 226 links rebuilt |
| Compiler command changed | All 460 tasks rebuilt |
| Clean entry | Removed both `build` and `.manny`; the next build completed all 460 tasks |

Two clean verification builds took 31.8 and 35.2 seconds on the test machine.
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
