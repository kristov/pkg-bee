# pkg-bee

Wouldn't it be cool if C libraries could be packaged and distributed like golang libraries? No? Oh well, here it is anyway. Enter pkg-bee!

Approach:

* Create a new git repository for your library
* Make a "Makefile" that does the right thing for "make" and "make clean"
* Running "make" should produce a static archive of your objects (\*.a)
* Put a special `package.pc` file in the root directory
* Push it somewhere (github I guess)

You can now run:

    pkg-bee build https://github.com/kristov/gl-matrix

Which will:

* Git clone `https://github.com/kristov/gl-matrix` into some location
* Enter that directory and execute "make"

Then you use it like `pkg-config` in your compilation target:

    target: target.c
        gcc -o target target.c $(pkg-bee --cflags --libs gl-matrix)

Which will execute `pkg-config --cflags --libs <package_dir>/package.pc` for that package.

## Where is the build dir?

By default it is located in `${HOME}/build` (it will not make that directory for you), or wherever the `PKG_BEE_BUILD` environment variable points to. So you can:

    PKG_BEE_BUILD=./build pkg-bee build https://github.com/kristov/gl-matrix
    PKG_BEE_BUILD=./build --cflags gl-matrix

For example.

## pkg-bee build

In build mode pkg-bee downloads the source and executes `make`.

    pkg-bee build <source>

Where `<source>` is assumed to be a git repository.

## package.pc

This file should be in the same format as `pkg-config` expects, for example:

    Name: gl-matrix
    Description: Library for matrix calculations
    Version: 0.1.0
    Conflicts:
    Libs: -L${pcfiledir} -l:gl-matrix.a
    Cflags: -I${pcfiledir}

This will tell `pkg-config` where your archive file can be found and how to link it. You can also add any other dynamic link flags required for your package to work.

## Why static?

There are several reasons why static linking is inferior to dynamic linking. For an interesting read see [this debian wiki page](https://wiki.debian.org/StaticLinking) on the topic. Still I find the toolchain is lacking something in the distribution of tiny libraries - perhaps why single-header libraries are a thing. Properly packaging dynamic libraries for distributions is [not easy](https://www.debian.org/doc/manuals/maint-guide/advanced.en.html).

## Gotchas

It's important to order the `--libs` correctly when compiling static archives. For example:

    gcc $(pkg-bee --cflags --libs gl-matrix) -o target target.c

Will not work, whereas:

    gcc -o target target.c $(pkg-bee --cflags --libs gl-matrix)

Should work. A good explanation can be found [here](https://eli.thegreenplace.net/2013/07/09/library-order-in-static-linking).

## Future

Probably no future. I will keep using this until I don't anymore.
