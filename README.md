# Microkit SDK builder

This is a convenience repository with tools and scripts intended to
help building the microkit SDK. The microkit SDK includes pre-built
ELF files of the seL4 microkernel, the system loader, and the monitor
task, for each supported board. It can be tedious to build it from a
development workstation.

## Submodules

The seL4 and microkit repositories modified to support x86_64 targets
are registered as submodules in this repository. After cloning, be
sure to issue a `git submodule update --init` to have the submodules
installed with the correct (tested) references.

## Docker

Two Docker images are provided:

- The `base` image includes all packages, libraries, and toolchains,
  required to build the microkit SDK for all supported target
  architectures and boards. The image weights about 4GB and should be
  built using the script `./docker/base/build`.

- The `workspace` image extends the `base` image to be used nearly
  transparently on a development workstation: it adds a user with
  identical name/UID/GID to the image, mounts the `$HOME` directory,
  and overlays an isolated `$HOME/.cache` directory. The image
  shouldbe built using the script `./docker/workspace/build` and used
  via the script `./docker/workspace/run`.

## Build script

A top-level `build` script uses the Docker images described above to
build the microkit SDK for all supported boards, or optionally for a
subset of boards. The first argument determines what to build:

- `docker`: build the two Docker images mentioned above.

- `sdk`: build the microkit SDK. Any following argument is taken as a
  specific board to build the SDK for. For instance `./build sdk`
  builds the SDK for all boards while `./build sdk x86_64_virt
  x86_64_supermicro` only builds the SDK for these two boards. For a
  list of supported board please have a look at the defition of
  `SUPPORTED_BOARDS` in the `./microkit/build_sdk.py` file. The SDK will
  be produced in `./microkit/release`.

- `example`: build the examples shipped with microkit. Not all boards
  provide examples, see `./microkit/example` for a list. The expected
  arguments are the name of the board and the name of the example for
  that board. The build products end up in `./microkit/tmp_build`.

- `clean`: remove all microkit build products.

Considering the long build times, it is advised to build the SDK only
for the board(s) of interest instead of building for all supported
boards.

## Testing

A convenience script `./microkit/example/x86_64_virt/sim` can be used to
start a QEMU instance configured to match the `x86_64_virt` target
board for microkit. The `./microkit/tmp_build/loader.img` should be
provided as first argument. All output happens on the serial port and
it is redirected to the console.

## TL;DR

```sh
$ ./build docker
$ ./build sdk x86_64_virt
$ ./build example x86_64_virt hello
$ ./microkit/example/x86_64_virt/sim ./microkit/tmp_build/loader.img
```
