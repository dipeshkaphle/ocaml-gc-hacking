# Ripping out the GC

## Prerequisites

- Have a global installation of a stable version of bdwgc. Instructions available at: https://github.com/ivmai/bdwgc?tab=readme-ov-file#building-and-installing)
- Opam
- Hyperfile
- Git
- Rust toolchain(to compile the allocator)

## Build

```bash
git clone --recurse-submodules https://github.com/prismlab/ocaml-gc-hacking
#Build the unchanged compiler
cd ocaml-gc-hacking/ocaml-4.14-unchanged
./configure --prefix=`pwd`/_install  --disable-naked-pointers
make -j
make install
#Build the version with hacked gc(verified)
cd ../ocaml-4.14-hacked-gc
./configure --disable-naked-pointers
make -C runtime -j ocamlrun ocamlrund
#Run tests
cd ../tests
make

#Build the version with bdwgc
cd ../ocaml-4.14-bdwgc
./configure --disable-naked-pointers
make -C runtime -j ocamlrun ocamlrund
#Run tests
cd ../tests
make
```

## Repositories

* `ocaml-4.14-unchanged` -- The unchanged OCaml 4.14 compiler. Used for
    compiling OCaml programs.
* `ocaml-4.14-hacked-gc` -- The hacked OCaml 4.14 compiler. This contains the
    modified `ocamlrun` used to run the modified GC(the verified one)
* `ocaml-4.14-bdwgc` --  Hacked Ocaml 4.14 runtime that uses Boehm GC
* `tests` -- contains some test, the directory has a bunch of Makefiles and README that should help with running the tests.


## Navigating tests directory

**NOTE**: You must have all the tools mentioned in [Prerequisites section](#prerequisites) and you should have completed all the steps given in [Build section](#build)

We have a bunch of targets set up in the `Makefile` in `tests` directory.

- **`bench` target** : Running `make bench` will build and run all some relevant examples with all three configuration(`verified-gc`, `bdwgc` and `unchanged OCaml GC`). 

- Targets of the format `<example>.bench` will also run the `<example>` with all three configuration.

- Targets of the format `<example>.csv` will run the `<example>` with all three configuration and some varying parameters and produce a csv with the same name, which has execution information.


## Misc Information

- The `MIN_EXPANSION_WORDSIZE` variable is used to tweak the size of heap(which
  will be `MIN_EXPANSION_WORDSIZE * 8 bytes`) for the allocator. We operate solely with that much heap and we will 
  be OOM if we can't find any space(after a verified GC). This can be seen in
  the `Makefile` in `tests` directory.
