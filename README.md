# C3 Language

C3 is a programming language that builds on the syntax and semantics of the C language, with the
goal of evolving it while still retaining familiarity for C programmers.

It's an evolution, not a revolution: the C-like language for programmers who like C.

Precompiled binaries for the following operating systems are available:

- Windows x64
  [download](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-windows.zip),
  [install instructions](#installing-on-windows-with-precompiled-binaries).
- Debian x64
  [download](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-linux.tar.gz),
  [install instructions](#installing-on-debian-with-precompiled-binaries).
- Ubuntu x86
  [download](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-ubuntu-22.tar.gz),
  [install instructions](#installing-on-ubuntu-with-precompiled-binaries).
- MacOS Arm64
  [download](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-macos.zip),
  [install instructions](#installing-on-macos-with-precompiled-binaries).
- OpenBSD x64
  [download](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-openbsd.tar.gz),
  [install instructions](#installing-on-openbsd-with-precompiled-binaries).

The manual for C3 can be found at [www.c3-lang.org](http://www.c3-lang.org).

![vkQuake](https://github.com/c3lang/c3c/blob/master/resources/images/vkQuake.png?raw=true)

Thanks to full ABI compatibility with C, it's possible to mix C and C3 in the same project with no
effort. As a demonstration, [vkQuake](https://github.com/c3lang/vkQuake) was compiled with a small
portion of the code converted to C3 and compiled with the c3c compiler.

For a non-curated list of user written projects and other resources, check out the
[c3-showcase](https://github.com/c3lang/c3-showcase).

## Design Principles

- Procedural "get things done"-type of language.
- Try to stay close to C - only change what's really necessary.
- C ABI compatibility and excellent C integration.
- Learning C3 should be easy for a C programmer.
- Data is inert.
- Avoid "big ideas" & the "more is better" fallacy.
- Introduce some higher level conveniences where the value is great.

C3 owes its inspiration to the [C2 language](http://c2lang.org): to iterate on top of C without
trying to be a whole new language.

### Example code

The following code shows [generic modules](https://c3-lang.org/generic-programming/generics/) (more
examples can be found at <https://c3-lang.org/language-overview/examples/>).

```cpp
module stack {Type};
// Above: the parameterized type is applied to the entire module.

struct Stack
{
    usz capacity;
    usz size;
    Type* elems;
}

// The type methods offers dot syntax calls, so this function can either be called
// `Stack.push(&my_stack, ...)` or `my_stack.push(...)`
fn void Stack.push(Stack* this, Type element)
{
    if (this.capacity == this.size)
    {
        this.capacity *= 2;
        if (this.capacity < 16) this.capacity = 16;
        this.elems = realloc(this.elems, Type.sizeof * this.capacity);
    }
    this.elems[this.size++] = element;
}

fn Type Stack.pop(Stack* this)
{
    assert(this.size > 0);
    return this.elems[--this.size];
}

fn bool Stack.empty(Stack* this)
{
    return !this.size;
}
```

Testing it out:

```cpp
import stack;

// Define our new types, the first will implicitly create a complete copy of the entire Stack
// module with `Type` set to `int`
alias IntStack = Stack {int};

// The second creates another copy with `Type` set to `double`
alias DoubleStack = Stack {double};

// If we had added `alias IntStack2 = Stack {int}` no additional copy would have been made, since
// we already have an parameterization of `Stack {int}` so it would be the same as declaring
// `IntStack2` an alias of `IntStack`

// Importing an external C function is straightforward.
// Here is an example of importing libc's `printf`:
extern fn int printf(char* format, ...);

fn void main()
{
    IntStack stack;
    // Note: C3 uses zero initialization by default so the above is equal to `IntStack stack = {};`

    stack.push(1);
    // The above can also be written `IntStack.push(&stack, 1);`

    stack.push(2);
    printf("pop: %d\n", stack.pop());   // > pop: 2
    printf("pop: %d\n", stack.pop());   // > pop: 1

    DoubleStack dstack;
    dstack.push(2.3);
    dstack.push(3.141);
    dstack.push(1.1235);
    printf("pop: %f\n", dstack.pop());  // > pop: 1.123500
}
```

### In what ways does C3 differ from C?

- No mandatory header files
- New semantic macro system
- Module based name spacing
- Slices !!!
- Operator overloading
- Compile time reflection
- Enhanced compile time execution
- Generics based on generic modules
- "Result"-based zero overhead error handling
- `defer` !!!
- Value methods
- Associated enum data
- No preprocessor
- Less undefined behaviour and added runtime checks in "safe" mode
- Limited operator overloading to enable userland dynamic arrays
- Optional pre and post conditions

### Current status

The current stable version of the compiler is **version 0.7.7**.

The upcoming 0.7.x releases will focus on expanding the standard library, fixing bugs and improving
compile time analysis.

If you have suggestions on how to improve the language, either [file an issue](https://github.com/c3lang/c3c/issues)
or discuss C3 on its dedicated [Discord](https://discord.gg/qN76R87).

The compiler is currently verified to compile on Linux, OpenBSD, Windows and MacOS.

#### Support matrix

| Platform           | Native `c3c` | Supported | Stack trace | Threads  | Sockets  | Inline asm |
|--------------------|--------------|-----------|-------------|----------|----------|------------|
| Win32 x64          | Yes          | Yes +     | Yes         | Yes      | Yes      | Yes*       |
| Win32 Aarch64      | Untested     | Untested  | Untested    | Untested | Untested | Yes*       |
| MacOS x64          | Yes          | Yes +     | Yes         | Yes      | Yes      | Yes*       |
| MacOS Aarch64      | Yes          | Yes +     | Yes         | Yes      | Yes      | Yes*       |
| iOS Aarch64        | No           | Untested  | Untested    | Yes      | Yes      | Yes*       |
| Linux x86          | Yes          | Yes       | Yes         | Yes      | Yes      | Yes*       |
| Linux x64          | Yes          | Yes       | Yes         | Yes      | Yes      | Yes*       |
| Linux Aarch64      | Yes          | Yes       | Yes         | Yes      | Yes      | Yes*       |
| Linux Riscv32      | Yes          | Yes       | Yes         | Yes      | Yes      | Untested   |
| Linux Riscv64      | Yes          | Yes       | Yes         | Yes      | Yes      | Untested   |
| ELF x86 nolibC     | No           | Untested  | No          | No       | No       | Yes*       |
| ELF x64 nolibC     | No           | Untested  | No          | No       | No       | Yes*       |
| ELF Aarch64 nolibC | No           | Untested  | No          | No       | No       | Yes*       |
| ELF Riscv64 nolibC | No           | Untested  | No          | No       | No       | Untested   |
| ELF Riscv32 nolibC | No           | Untested  | No          | No       | No       | Untested   |
| FreeBSD x86        | Untested     | Untested  | No          | Yes      | Untested | Yes*       |
| FreeBSD x64        | Untested     | Untested  | No          | Yes      | Untested | Yes*       |
| NetBSD x86         | Untested     | Untested  | No          | Yes      | Untested | Yes*       |
| NetBSD x64         | Untested     | Untested  | No          | Yes      | Untested | Yes*       |
| OpenBSD x86        | Untested     | Untested  | No          | Yes      | Untested | Yes*       |
| OpenBSD x64        | Yes*         | Yes       | Yes*        | Yes      | Untested | Yes*       |
| MCU x86            | No           | Untested  | No          | No       | No       | Yes*       |
| Wasm32             | No           | Yes       | No          | No       | No       | No         |
| Wasm64             | No           | Untested  | No          | No       | No       | No         |

_+ Also supports cross-compilation_<br>
_* Inline asm is still a work in progress_<br>
_* OpenBSD 7.7 is the only tested version_<br>
_* OpenBSD has limited stacktrace, needs to be tested further_

More platforms will be supported in the future.

#### What can you help with?

- If you wish to contribute with ideas, please file issues or discuss on Discord.
- Interested in contributing to the stdlib? Please get in touch on Discord.
- Compilation instructions for other Linux and Unix variants are appreciated.
- Would you like to contribute bindings to some library? It would be nice to have support for SDL,
  Raylib and more.
- Build something with C3 and show it off and give feedback. The language is still open for
  significant tweaks.
- Start work on the C -> C3 converter which takes C code and does a "best effort" to translate it
  to C3. The first version only needs to work on C headers.
- Do you have some specific area you have deep knowledge of and could help make C3 even better at
  doing? File or comment on issues.

### Installing

This installs the latest prerelease build, as opposed to the latest released version.

#### Installing on Windows with precompiled binaries

1. Download the [zip file](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-windows.zip)
   ([debug version](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-windows-debug.zip))
2. Unzip executable and standard library.
3. If you don't have Visual Studio 17 on your system, you can either install it manually or run the
   `msvc_build_libraries.py` script to do so.
4. Run `c3c.exe`.

#### Installing on Windows with the install script

Open a PowerShell terminal (you may need to run it as an administrator) and run the following command:

```ps
iwr -useb https://raw.githubusercontent.com/c3lang/c3c/refs/heads/master/install/install.ps1 | iex
```

The script will inform you once the installation is successful and add the `%USERPROFILE%\.c3`
directory to your PATH, which will allow you to run the `c3c` command from any location.

You can choose another version with option `C3_VERSION`. For example, you can force the
installation of the 0.7.4 version:

```ps
$env:C3_VERSION = '0.7.4'
$script = {
    iwr -useb https://raw.githubusercontent.com/c3lang/c3c/refs/heads/master/install/install.ps1 | iex
}
PowerShell -ExecutionPolicy Bypass -Command $script
```

If you don't have Visual Studio 17 on your system, you can either install it manually or run the
`msvc_build_libraries.py` script to do so.

#### Installing on Debian with precompiled binaries

1. Download the [tar file](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-linux.tar.gz)
   ([debug version](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-linux-debug.tar.gz))
2. Unpack executable and standard library.
3. Run `./c3c`.

#### Installing on Debian with the install script

Open a terminal and run the following command:

```sh
curl -fsSL https://raw.githubusercontent.com/c3lang/c3c/refs/heads/master/install/install.sh | bash
```

The C3 compiler will be installed, and the script will also update your `~/.bashrc` to include
`~/.c3` in your PATH, allowing you to invoke the c3c command from anywhere. You might need to
restart your terminal or source your shell for the changes to take effect.

You can choose another version with option `C3_VERSION`.
For example, you can force the installation of the 0.7.4 version:

```sh
curl -fsSL https://raw.githubusercontent.com/c3lang/c3c/refs/heads/master/install/install.sh |
    C3_VERSION=0.7.4 bash
```

#### Installing on Ubuntu with precompiled binaries

1. Download [tar file](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-ubuntu-22.tar.gz)
   ([debug version](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-ubuntu-22-debug.tar.gz))
2. Unpack executable and standard library.
3. Run `./c3c`.

#### Installing on MacOS with precompiled binaries

1. Make sure you have XCode with command line tools installed.
2. Download the [zip file](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-macos.zip)
   ([debug version](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-macos-debug.zip))
3. Unzip executable and standard lib.
4. Run `./c3c`.

_*Note: there is a known issue with debug symbol generation on MacOS 13, see
[issue #1086](https://github.com/c3lang/c3c/issues/1086)_

#### Installing on OpenBSD with precompiled binaries

1. Download [tar file](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-openbsd.tar.gz)
   ([debug version](https://github.com/c3lang/c3c/releases/download/latest-prerelease/c3-openbsd-debug.tar.gz))
2. Unpack executable and standard lib.
3. Run `./c3c`.

_\*Note: this is specifically for OpenBSD 7.7, running it on any other version is prone to ABI breaks_

#### Installing on Arch Linux

Arch includes c3c in the official 'extra' repo. It can be easily installed the usual way:

```sh
sudo pacman -S c3c
# or yay -S c3c
# or paru -S c3c
# or aura -A c3c
```

There is also an AUR package for the c3c compiler : [c3c-git](https://aur.archlinux.org/packages/c3c-git).

You can use your AUR package manager:

```sh
yay -S c3c-git
# or paru -S c3c-git
# or aura -A c3c-git
```

Or clone it manually:

```sh
git clone https://aur.archlinux.org/c3c-git.git
cd c3c-git
makepkg -si
```

#### Installing via Nix

You can access `c3c` via [flake.nix](./flake.nix), which will contain the latest commit of the
compiler. To add `c3c` to your `flake.nix`, do the following:

```nix
{
    inputs = {
        nixpkgs.url = "github:nixos/nixpkgs?ref=nixpkgs-unstable";
        flake-utils.url = "github:numtide/flake-utils";
        c3c.url = "github:c3lang/c3c";
        # Those are desired if you don't want to copy extra nixpkgs
        c3c.inputs = {
            nixpkgs.follows = "nixpkgs";
            flake-utils.follows = "flake-utils";
        };
    };

    outputs = { self, ... } @ inputs: inputs.flake-utils.lib.eachDefaultSystem (system:
        let
            pkgs = import inputs.nixpkgs { inherit system; };
            c3c = inputs.c3c.packages.${system}.c3c;
        in
        {
            devShells.default = pkgs.mkShell {
                buildInputs = [
                    pkgs.c3c
                ];
            };
        }
    );
}
```

#### Installing on Gentoo

`c3c` is available in the [Gentoo GURU overlay](https://wiki.gentoo.org/wiki/Project:GURU).

Enable and sync the GURU repository (if not already done):

```sh
sudo eselect repository enable guru
sudo emaint sync -r guru
```

Install `c3c` with:

```sh
sudo emerge -av dev-lang/c3c
```

- The compiler binary is installed to `/usr/bin/c3c`.
- The standard library is installed to `/usr/lib/c3`.

_*Note: for Gentoo-specific issues, please use the [Gentoo Bugzilla](https://bugs.gentoo.org/) (Product: GURU)_

#### Building via Docker

You can build `c3c` using an Ubuntu container. By default, the script will build through Ubuntu
22.04. You can specify the version by passing the `UBUNTU_VERSION` environment variable.

```sh
UBUNTU_VERSION=22.04 ./build-with-docker.sh
```

See the `build-with-docker.sh` script for more information on other configurable environment variables.

#### Installing on Windows using Scoop

c3c is included in 'Main' bucket.

```sh
scoop install c3
```

### Getting started with a "hello world"

Create a `main.c3` file with:

```cpp
module hello_world;
import std::io;

fn void main()
{
   io::printn("Hello, world!");
}
```

Make sure you have the standard libraries at either `../lib/std/` or `/lib/std/`.

Then run

```sh
c3c compile main.c3
```

The generated binary will by default be named after the module that contains the main
function. In our case that is `hello_world`, so the resulting binary will be
called `hello_world` or `hello_world.exe`depending on platform.

### Compiling

#### Compiling on Windows

1. Make sure you have Visual Studio 17 2022 installed or alternatively install the "Buildtools for
   Visual Studio" (<https://aka.ms/vs/17/release/vs_BuildTools.exe>) and then select "Desktop
   development with C++"
2. Install CMake
3. Clone the C3C github repository: `git clone https://github.com/c3lang/c3c.git`
4. Enter the C3C directory: `cd c3c`.
5. Set up the CMake build: `cmake --preset windows-vs-2022-release`
6. Build: `cmake --build --preset windows-vs-2022-release`

You should now have a `c3c` executable in `build\Release`.

You can try it out by running some sample code: `c3c.exe compile ../../resources/examples/hash.c3`

Building `c3c` using Visual Studio Code is also supported when using the `CMake Tools` extension.
Simply select the `Windows x64 Visual Studio 17 2022` configure preset and build.

_*Note: if you run into linking issues when building, make sure that you are using the latest
version of VS17_

#### Compiling on Windows (Debug)

Debug build requires a different set of LLVM libraries to be loaded for which a separate CMake
configuration is used to avoid conflicts.

1. Configure: `cmake --preset windows-vs-2022-debug`
2. Build: `cmake --build --preset windows-vs-2022-debug`

You should now have a `c3c` executable in `build-debug\Debug`.

#### Compiling on Ubuntu 24.04 LTS

1. Make sure you have a C compiler that handles C11 and a C++ compiler, such as GCC or Clang. Git
   also needs to be installed.
2. Install build dependencies:

    ```sh
    sudo apt install \
        clang \
        cmake \
        git \
        liblld-18 \
        liblld-dev \
        libllvm18 \
        libpolly-18-dev \
        llvm \
        llvm-dev \
        llvm-runtime \
        zlib1g \
        zlib1g-dev
    ```

    If you're using Ubuntu 25.04, also install `libpolly-20-dev`.

3. Clone the C3C github repository: `git clone https://github.com/c3lang/c3c.git`
4. Enter the C3C directory `cd c3c`.
5. Set up CMake build: `cmake -B build -S .`
6. Build: `cmake --build build`
7. Change directory to the build directory `cd build`

You should now have a `c3c` executable.

You can try it out by running some sample code: `./c3c compile ../resources/examples/hash.c3`

#### Compiling on Void Linux

1. As root, ensure that all project dependencies are installed:

    ```sh
    xbps-install \
        cmake \
        git \
        libcurl-devel \
        libxml2-devel \
        libzstd-devel \
        lld17-devel \
        llvm17 \
        llvm17-devel \
        ncurses-devel \
        zlib-devel
    ```

2. Clone the C3C repository: `git clone --depth=1 https://github.com/c3lang/c3c.git`
3. Enter the directory: `cd c3c`
4. Create the CMake build cache: `cmake -B build -S .`
5. Build: `cmake --build build`
6. Enter the build directory: `cd build`

Your c3c executable should have compiled properly. You may want to test it: `./c3c compile ../resources/examples/hash.c3`
For a system-wide installation, run the following as root: `cmake --install .`

#### Compiling on Fedora

1. Install required project dependencies: `dnf install cmake clang git llvm llvm-devel lld lld-devel ncurses-devel`
2. Optionally, install additional dependencies: `dnf install libcurl-devel zlib-devel libzstd-devel libxml2-devel libffi-devel`
3. Clone the C3C repository: `git clone https://github.com/c3lang/c3c.git`
    - If you only need the latest commit, you may want to make a shallow clone: `git clone https://github.com/c3lang/c3c.git --depth=1`
4. Enter the C3C directory: `cd c3c`
5. Create the CMake build cache. The Fedora repositories provide `.so` libraries for lld, so you need to set the C3_LINK_DYNAMIC flag: `cmake -B build -S . -DC3_LINK_DYNAMIC=1`
6. Build the project: `cmake --build build`
7. Enter the build directory: `cd build`

The c3c binary should be created in the build directory. You can try it out by running some sample code: `./c3c compile ../resources/examples/hash.c3`

#### Compiling on Arch Linux

1. Install required project dependencies:

    ```sh
    sudo pacman -S --needed \
        clang \
        cmake \
        curl \
        git \
        libedit \
        libxml2 \
        lld \
        llvm \
        llvm-libs
    ```

2. Clone the C3C repository: `git clone https://github.com/c3lang/c3c.git`
    - If you only need the latest commit, you may want to make a shallow clone: `git clone https://github.com/c3lang/c3c.git --depth=1`
3. Enter the C3C directory: `cd c3c`
4. Create the CMake build cache:

    ```sh
    cmake -B build \
        --install-prefix $HOME/.local \
        -DC3_LINK_DYNAMIC=ON \
        -DCMAKE_BUILD_TYPE=Release
    ```

5. Build the project: `cmake --build build`. After compilation, the `c3c` binary will be located in
   the `build` directory. You can test it by compiling an example: `./build/c3c compile
   resources/examples/ls.c3`.

6. To install the compiler your user profile: `sudo cmake --install build`

#### Compiling on NixOS

1. Enter nix shell, by typing `nix develop` in root directory
2. Configure cmake via `cmake . -Bbuild $=C3_CMAKE_FLAGS`. Note: passing `C3_CMAKE_FLAGS` is needed in due to generate `compile_commands.json` and find missing libs.
4. Build it `cmake --build build`
5. Test it out: `./build/c3c -V`
6. If you use `clangd` lsp server for your editor, it is recommended to make a symbolic link to `compile_command.json` in the root: `ln -s ./build/compile_commands.json compile_commands.json`

#### Compiling on OS X using Homebrew

1. Install [Homebrew](https://brew.sh/)
2. Install LLVM 17+: `brew install llvm`
3. Install lld: `brew install lld`
4. Install CMake: `brew install cmake`
5. Clone the C3C github repository: `git clone https://github.com/c3lang/c3c.git`
6. Enter the C3C directory `cd c3c`.
7. Set up CMake build for debug: `cmake -B build -S .`
8. Build: `cmake --build build`
9. Change directory to the build directory `cd build`

#### Compiling on other Linux / Unix variants

1. Install CMake.
2. Install or compile LLVM and LLD libraries (version 17+ or higher)
3. Clone the C3C github repository: `git clone https://github.com/c3lang/c3c.git`
4. Enter the C3C directory `cd c3c`.
5. Set up CMake build for debug: `cmake -B build -S .`. At this point you may need to manually
   provide the link path to the LLVM CMake directories, e.g. `cmake -B build -S . -DLLVM_DIR=/usr/local/opt/llvm/lib/cmake/llvm/`
6. Build: `cmake --build build`
7. Change directory to the build directory `cd build`

_*Note on compiling for Linux/Unix/MacOS: to be able to fetch vendor libraries, `libcurl` is needed.
The CMake script should detect it if it is available but functionality is non-essential
and it is perfectly fine to use the compiler without it._

### Licensing

Unless specified otherwise, the code in this repository is MIT licensed.
The exception is the compiler source code (the source code under `src`),
which is licensed under LGPL 3.0.

This means you are free to use all parts of standard library,
tests, benchmarks, grammar, examples and so on under the MIT license, including
using those libraries and tests if your build your own C3 compiler.

### Editor plugins

Editor plugins can be found at <https://github.com/c3lang/editor-plugins>.

### Contributing unit tests

1. Write the test, either adding to existing test files in `/test/unit/` or add
   a new file. (If testing the standard library, put it in the `/test/unit/stdlib/` subdirectory).
2. Make sure that the test functions have the `@test` attribute.
3. Run tests and see that they pass. (Recommended settings: `c3c compile-test -O0 test/unit`.
   - in this example `test/unit/` is the relative path to the test directory, so adjust as required)
4. Make a pull request for the new tests.

## Thank yous

A huge **THANK YOU** goes out to all contributors and sponsors.

A special thank you to sponsors [Zack Puhl](https://github.com/NotsoanoNimus) and
[konimarti](https://github.com/konimarti) for going the extra mile.

And honorable mention goes to past sponsors:
[Ygor Pontelo](https://github.com/ygorpontelo), [Simone Raimondi](https://github.com/SRaimondi),
[Jan Válek](https://github.com/jan-valek), [Pierre Curto](https://github.com/pierrec),
[Caleb-o](https://github.com/Caleb-o) and [devdad](https://github.com/devdad)

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=c3lang/c3c&type=Date)](https://www.star-history.com/#c3lang/c3c&Date)
