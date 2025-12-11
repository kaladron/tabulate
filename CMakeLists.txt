cmake_minimum_required(VERSION 3.8)
project(tabulate VERSION 1.5.0 LANGUAGES CXX)
option(tabulate_BUILD_TESTS OFF)
option(SAMPLES "Build Samples" OFF)
option(BUILD_MODULE "Build C++20 module (requires CMake 3.28+ and C++20 compiler with modules support)" OFF)

set(TABULATE_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Versioning
# ===========

# Project version
file(STRINGS "${TABULATE_INCLUDE_DIR}/tabulate/tabulate.hpp" tabulate_version_defines
     REGEX "#define TABULATE_VERSION_(MAJOR|MINOR|PATCH)")
foreach(ver ${tabulate_version_defines})
    if(ver MATCHES "#define TABULATE_VERSION_(MAJOR|MINOR|PATCH) +([^ ]+)$")
        set(TABULATE_VERSION_${CMAKE_MATCH_1} "${CMAKE_MATCH_2}" CACHE INTERNAL "")
    endif()
endforeach()
set(TABULATE_VERSION
    ${TABULATE_VERSION_MAJOR}.${TABULATE_VERSION_MINOR}.${TABULATE_VERSION_PATCH})
message(STATUS "tabulate version: v${TABULATE_VERSION}")

if(MSVC)
  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  endif()
elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -pedantic")
endif()

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

add_library(tabulate INTERFACE)
add_library(tabulate::tabulate ALIAS tabulate)

if (USE_CPP17)
  target_compile_features(tabulate INTERFACE cxx_std_17)
  message(STATUS "Using C++17")
else()
  target_compile_features(tabulate INTERFACE cxx_std_11)
  message(STATUS "Using C++11")
endif()

target_include_directories(tabulate INTERFACE
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include>)

if( SAMPLES )
  add_subdirectory(samples)
endif()

# C++20 Module Support
# ====================
# Only available when BUILD_MODULE is ON and CMake >= 3.28 with C++20 support
# Requires Ninja, Ninja Multi-Config, or Visual Studio >= 17.4 generator
# Note: GCC may require additional setup for module scanning; Clang >= 16 is recommended
if(BUILD_MODULE)
  if(CMAKE_VERSION VERSION_LESS "3.28")
    message(WARNING "C++20 modules require CMake 3.28 or higher. Module target will not be built.")
  elseif(NOT (CMAKE_GENERATOR MATCHES "Ninja" OR CMAKE_GENERATOR MATCHES "Visual Studio"))
    message(WARNING "C++20 modules require Ninja or Visual Studio generator. Current generator '${CMAKE_GENERATOR}' is not supported. Module target will not be built.")
  else()
    # Check for C++20 support
    include(CheckCXXCompilerFlag)
    check_cxx_compiler_flag("-std=c++20" COMPILER_SUPPORTS_CXX20)
    if(NOT COMPILER_SUPPORTS_CXX20)
      check_cxx_compiler_flag("/std:c++20" COMPILER_SUPPORTS_CXX20_MSVC)
      if(COMPILER_SUPPORTS_CXX20_MSVC)
        set(COMPILER_SUPPORTS_CXX20 TRUE)
      endif()
    endif()

    if(COMPILER_SUPPORTS_CXX20)
      message(STATUS "Building C++20 module")

      # Enable module scanning
      set(CMAKE_CXX_SCAN_FOR_MODULES ON)

      add_library(tabulate_module)
      target_sources(tabulate_module
        PUBLIC
          FILE_SET CXX_MODULES FILES
          ${CMAKE_CURRENT_SOURCE_DIR}/include/tabulate/tabulate.cppm
      )
      target_compile_features(tabulate_module PUBLIC cxx_std_20)
      target_include_directories(tabulate_module PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
      )
      add_library(tabulate::tabulate_module ALIAS tabulate_module)
    else()
      message(WARNING "C++20 is not supported by the compiler. Module target will not be built.")
    endif()
  endif()
endif()

configure_package_config_file(tabulateConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/tabulateConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tabulate)

write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
                                 VERSION ${TABULATE_VERSION}
                                 COMPATIBILITY AnyNewerVersion)

install(TARGETS tabulate EXPORT tabulateTargets)
install(EXPORT tabulateTargets
        FILE tabulateTargets.cmake
        NAMESPACE tabulate::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tabulate)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/tabulateConfig.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/tabulate)
install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/tabulate
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        USE_SOURCE_PERMISSIONS
        PATTERN "*.hpp"
        PATTERN "*.cppm")
