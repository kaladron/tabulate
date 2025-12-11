# Using tabulate

This guide covers different ways to integrate tabulate into your C++ project.

## Table of Contents

- [Header-Only Usage](#header-only-usage)
- [CMake Integration](#cmake-integration)
  - [FetchContent](#fetchcontent)
  - [find_package](#find_package)
- [C++20 Modules](#c20-modules)
  - [Building with Modules](#building-with-modules)
  - [Using Modules with FetchContent](#using-modules-with-fetchcontent)
- [Single Header](#single-header)

## Header-Only Usage

tabulate is a header-only library. Simply add `include/` to your include directories:

```cpp
#include <tabulate/table.hpp>
using namespace tabulate;

int main() {
    Table table;
    table.add_row({"Name", "Value"});
    table.add_row({"Hello", "World"});
    std::cout << table << std::endl;
}
```

## CMake Integration

### FetchContent

The recommended way to use tabulate with CMake is via `FetchContent`:

```cmake
include(FetchContent)

FetchContent_Declare(
  tabulate
  GIT_REPOSITORY https://github.com/p-ranav/tabulate.git
  GIT_TAG master
  GIT_SHALLOW TRUE
  EXCLUDE_FROM_ALL
)

FetchContent_MakeAvailable(tabulate)

add_executable(my_app main.cpp)
target_link_libraries(my_app PRIVATE tabulate::tabulate)
```

### find_package

If you have installed tabulate system-wide:

```cmake
find_package(tabulate REQUIRED)

add_executable(my_app main.cpp)
target_link_libraries(my_app PRIVATE tabulate::tabulate)
```

## C++20 Modules

tabulate provides C++20 module support, allowing you to use `import tabulate;` instead of header includes. The module is built when `USE_CPP20` is enabled.

### Requirements

- CMake 3.28 or higher
- `USE_CPP20=ON` CMake option
- C++20 compatible compiler (Clang 16+ recommended)
- Generator that supports C++20 modules (Ninja or Visual Studio 17.4+)

### Building with Modules

Enable C++20 mode to build the module:

```bash
cmake -B build -G Ninja -DUSE_CPP20=ON
cmake --build build
```

To disable module building while still using C++20, set `TABULATE_BUILD_MODULE=OFF`:

```bash
cmake -B build -G Ninja -DUSE_CPP20=ON -DTABULATE_BUILD_MODULE=OFF
```

### Using Modules with FetchContent

Set `USE_CPP20` before calling `FetchContent_MakeAvailable`:

```cmake
cmake_minimum_required(VERSION 3.28)
project(my_project CXX)

include(FetchContent)

FetchContent_Declare(
  tabulate
  GIT_REPOSITORY https://github.com/p-ranav/tabulate.git
  GIT_TAG master
  GIT_SHALLOW TRUE
  EXCLUDE_FROM_ALL
)

# Enable C++20 mode before making tabulate available
set(USE_CPP20 ON CACHE BOOL "" FORCE)

FetchContent_MakeAvailable(tabulate)

add_executable(my_app main.cpp)
target_link_libraries(my_app PRIVATE tabulate::tabulate_module)
```

Then in your code:

```cpp
import tabulate;
#include <iostream>

int main() {
    tabulate::Table table;
    table.add_row({"Name", "Value"});
    table.add_row({"Hello", "World"});
    table.format().font_style({tabulate::FontStyle::bold});
    std::cout << table << std::endl;
}
```

**Note**: The C++20 module requires the Ninja generator:

```bash
cmake -B build -G Ninja
cmake --build build
```

### Exported Types

The module exports the following types from the `tabulate` namespace:

- `Table` - Main table class
- `FontAlign` - Font alignment enum
- `FontStyle` - Font style enum
- `Color` - Color enum
- `Format` - Format settings class
- `RowStream` - Stream-based row builder

## Single Header

A single header version is available in `single_include/`. Simply copy `single_include/tabulate/table.hpp` to your project:

```cpp
#include "table.hpp"
using namespace tabulate;

int main() {
    Table table;
    table.add_row({"Single", "Header"});
    std::cout << table << std::endl;
}
```
