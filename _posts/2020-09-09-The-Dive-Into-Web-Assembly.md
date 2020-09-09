---
title: "The Dive Into Web Assembly"
published: true
---

# Background

I was first introduced to web assembly, also known as wasm, by [Kevin
Helfert](https://twitter.com/KevinHelfert). I at first made fun of him for his
excitement to this newer technology never knowing that someday it would help me
bring bread to the table. Well, that day has happened recently. 

# The Problem

I am working with a client who has a C++ library used to solve finite element
models. But, their clients wanted the ability to use these applications out in
the field. We were presented with a few options:
- Develop individual applications for each requested platform
- Use tools like Flutter or React Native to develop more multiple platforms at once
- Use wasm to allow us to deliver that application as a PWA.

Thankfully, I got the go ahead to experiment with this newer technology that
compiled our C++ libraries into wasm. 

This C++ library uses CMake as the build system. This means that it should be
easier to change our toolchains to build wasm. 

## The Solution

## Building C++ Libraries

Enter [Emscripten](https://emscripten.org/index.html). This is the project that
allows for LLVM, the intermediate representation associated with Clang based
compilers, to be compiled into wasm. This is a genius approach to compiling for
a new, different platform. I was accustomed to LLVM having built a compiler for
my senior project as a requirement for my Bachelor's degree, but I never thought
about using it to generate output like wasm. Now that I have completed this
project, it makes a lot of sense and certain elements of LLVM have clicked.

The approach is simple in theory, a little bit more difficult in practice:
- Use `emcmake cmake ...` to generate build files
- Use `emmake make ...` to build the project

This takes care of most of the toolchain issues because it changes gcc/g++ to
emcc/em++ and thus the webasm is taken care of. But, an issue arises when you
are using libraries within your libraries. A good case is
[spdlog](https://github.com/gabime/spdlog). To be able to compile everything to
C++, all libraries need to be built from source. While this isn't a showstopper,
it is an annoyance at first. Thankfully CMake can come to the rescue:

```
### START spdlog SETUP
include(ExternalProject)
set(EXTERNAL_INSTALL_LOCATION ${CMAKE_BINARY_DIR}/external)
ExternalProject_Add(project_spdlog
  GIT_REPOSITORY    https://github.com/gabime/spdlog.git
  GIT_TAG           v1.8.0
  CMAKE_ARGS        -DCMAKE_INSTALL_PREFIX=${EXTERNAL_INSTALL_LOCATION}
  BUILD_BYPRODUCTS   ${EXTERNAL_INSTALL_LOCATION}/lib/libspdlog.a
)


add_library(spdlog STATIC IMPORTED GLOBAL)
set_property(TARGET spdlog PROPERTY IMPORTED_LOCATION ${EXTERNAL_INSTALL_LOCATION}/lib/libspdlog.a)
add_dependencies(spdlog project_spdlog)


add_dependencies(${projectName} spdlog)
```

Having the ability to automatically build and include `spdlog` is super useful.
I will be adding my CMake project template to github soon. This will allow at
least myself to create C++ projects easier and faster since the frameworks will
be setup.

## Adding the JavaScript Interface

Now it is possible to use the wasm directly, but I'm a big fan of using
interfaces for those interactions. Enter
[Embind](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/embind.html).
This is one of the two options for building Javascript interfaces. By describing
the public interface of your library and compiling everything together, the
resulting `.js` and `.wasm` files can be directly added to a website and used. I
recommend using the modular options so that it can be added into React websites
easier:
```
em++ --bind -o <projectName>.js wasm.cpp lib<projectName>.a external/lib/lib* -s MODULARIZE=1 -s 'EXPORT_NAME="create<projectName>"'
```
If you need import statements and such, they can be included in that command.
But that single command will take all the libraries and that `wasm.cpp`
interface description file and output the needed files to use on the web.