# Simplified build system

This directory contains the simplified build system. You can use this to
compile, and cross compile, Measurement Kit and its dependencies. Specifically
the following are the _packages_ that you can compile:

- `geoip`: [geoip](https://github.com/maxmind/geoip-api-c)
- `libressl`: [libressl](https://github.com/libressl-portable)
- `libevent`: [libevent](https://github.com/libevent/libevent)
- `c-ares`: [c-ares](https://github.com/c-ares/c-ares)
- `mk`: [MK](https://github.com/measurement-kit/measurement-kit) itself

If needed, the build system will fetch the required sources and possibly
apply patches. In such case, sources will be downloaded in

```
./MK_BUILD/<os>/<arch>/<package>/<package-sources-dir>/
```

Artifacts (i.e. libraries and headers) will be installed in

```
./MK_DIST/<os>/<arch>/<package>/
```

(Paths are relative to the top level directory of MK sources.)

If a package needs another package, you need to compile that other package
first (or to populate appropriately `./MK_DIST/<os>/<arch>/<package>/` using
the artifacts generated by a previous build).

All the scripts except `build.bat` can actually be called from everywhere.

## General: passing flags to make and to compiler

In all types of build, except for Windows, we use `make` to orchestrate
the build. Unlike the previous build system, we do not support passing flags
to make directly using a specific environment variable. However, `make`
should honour `MAKEFLAGS`, so you can use that.

Likewise, you can specify `CPPFLAGS`, `CFLAGS`, `CXXFLAGS` and `LDFLAGS`
flags in environment variables, to override the flags that we pass to
the compiler. The flags you specify will be appended after the flags we
specify, giving you the opportunity to override such flags.

## Build on Linux / macOS

Just call the script named after the package to build. E.g.:

```
./script/build/libressl
```

In addition to Unix essential commands, you'll need:

- make
- autoconf and automake
- g++ or clang (must be C++14-capable)
- libstdc++ or libc++ (again must be C++14-capable)
- git
- libtool
- wget
- gpg

## Cross compile for iOS

We assume that you have a macOS system with Xcode installed and the command
line developer tools installed.

```
./script/build/cross-ios <os> <arch> ./script/build/<package>
```

To see available `<os>` and `<arch>` values, run `cross-ios` without arguments.

## Cross compile for Android

We assume that you have the Native Development Kit (NDK) installed. The simplest
and recommended way to get the NDK is to install it along with Android Studio.

Create a standalone Android toolchain using the `android-toolchain` script:

```
./script/build/android-toolchain <ndk-bundle-path> <arch>
```

Run the script without arguments to see all the available `<arch>` options.
The toolchain will be created in:

```
./MK_BUILD/android/toolchain/<arch>/
```

Then cross compile using:

```
./script/build/cross-android <arch> ./script/build/<package>
```

## Compile for Windows

### MSYS2

We assume that you have installed [MSYS2](https://www.msys2.org/) on a
Windows system (we generally use Windows 10).

For the required packages and the exact commands to run, it is best
that you refer to the build script that we use for appveyor (see
[.ci/appveyor](../../.ci/appveyor)).

A possible source of troubles is that MSYS2 provides you with three
shells: normal, 32 bit, and 64 bit development. If you're in the 64 bit
shell, the `gcc` compiler reported by which should be like:

```
$ which gcc
/mingw64/bin/gcc
```

where the important part is `/mingw64/bin` as opposed to `/usr/bin`.

If you're not in such shell, set your `PATH` properly (this is done by
the script that we use for building on appveyor).

For building dependencies, rather than MK, just install the required
packages and then, inside the proper shell as mentioned above, run
`./script/build/<package>` (i.e. like when you are on Unix).

### Visual Studio

We assume that:

1. you have installed Visual Studio 2017 Community

2. you have installed CMake and opted to have `cmake.exe` inside your path
   during the install process (important, as that is not the default
   choice)

3. you have installed Ubuntu and `ubuntu.exe` is available from your
   path (that should be the case), and you have installed into Ubuntu
   on Windows all the Linux packages mentioned above when discussing
   building for Linux or macOS

4. you are typing inside the x32 or x64 Developer Command prompt

5. you `chdir`-ed into the top-level directory of MK sources

```
cd script\build
build.bat <package>
```

This way of building MK is currently not working because there are
parts of MK that do not work with Microsoft compiler.