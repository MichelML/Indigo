########################################################################
# Note: CMake support is community-based. The maintainers do not use CMake
# internally.
#
# CMake build script for Google Test.
#
# To run the tests for Google Test itself on Linux, use 'make test' or
# ctest.  You can select which tests to run using 'ctest -R regex'.
# For more options, run 'ctest --help'.

# When other libraries are using a shared version of runtime libraries,
# Google Test also has to use one.
option(
  gtest_force_shared_crt
  "Use shared (DLL) run-time lib even when Google Test is built as static lib."
  ON)

option(gtest_build_tests "Build all of gtest's own tests." OFF)

option(gtest_build_samples "Build gtest's sample programs." OFF)

option(gtest_disable_pthreads "Disable uses of pthreads in gtest." OFF)

option(
  gtest_hide_internal_symbols
  "Build gtest with internal symbols hidden in shared libraries."
  OFF)

# Defines pre_project_set_up_hermetic_build() and set_up_hermetic_build().
include(cmake/hermetic_build.cmake OPTIONAL)

if (COMMAND pre_project_set_up_hermetic_build)
  pre_project_set_up_hermetic_build()
endif()

########################################################################
#
# Project-wide settings

# Name of the project.
#
# CMake files in this project can refer to the root source directory
# as ${gtest_SOURCE_DIR} and to the root binary directory as
# ${gtest_BINARY_DIR}.
# Language "C" is required for find_package(Threads).

# Project version:
cmake_policy(SET CMP0048 NEW)
project(gtest LANGUAGES CXX C)
cmake_minimum_required(VERSION 3.4)

if (POLICY CMP0063) # Visibility
  cmake_policy(SET CMP0063 NEW)
endif (POLICY CMP0063)

if (COMMAND set_up_hermetic_build)
  set_up_hermetic_build()
endif()

# These commands only run if this is the main project
if(CMAKE_PROJECT_NAME STREQUAL "gtest" OR CMAKE_PROJECT_NAME STREQUAL "googletest-distribution")

  # BUILD_SHARED_LIBS is a standard CMake variable, but we declare it here to
  # make it prominent in the GUI.
  option(BUILD_SHARED_LIBS "Build shared libraries (DLLs)." OFF)

else()

  mark_as_advanced(
    gtest_force_shared_crt
    gtest_build_tests
    gtest_build_samples
    gtest_disable_pthreads
    gtest_hide_internal_symbols)

endif()


if (gtest_hide_internal_symbols)
  set(CMAKE_CXX_VISIBILITY_PRESET hidden)
  set(CMAKE_VISIBILITY_INLINES_HIDDEN 1)
endif()

# Define helper functions and macros used by Google Test.
include(cmake/internal_utils.cmake)

config_compiler_and_linker()  # Defined in internal_utils.cmake.

# Create the CMake package file descriptors.
if (INSTALL_GTEST)
  include(CMakePackageConfigHelpers)
  set(cmake_package_name GTest)
  set(targets_export_name ${cmake_package_name}Targets CACHE INTERNAL "")
  set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated" CACHE INTERNAL "")
  set(cmake_files_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/${cmake_package_name}")
  set(version_file "${generated_dir}/${cmake_package_name}ConfigVersion.cmake")
  write_basic_package_version_file(${version_file} VERSION ${GOOGLETEST_VERSION} COMPATIBILITY AnyNewerVersion)
  install(EXPORT ${targets_export_name}
    NAMESPACE ${cmake_package_name}::
    DESTINATION ${cmake_files_install_dir})
  set(config_file "${generated_dir}/${cmake_package_name}Config.cmake")
  configure_package_config_file("${gtest_SOURCE_DIR}/cmake/Config.cmake.in"
    "${config_file}" INSTALL_DESTINATION ${cmake_files_install_dir})
  install(FILES ${version_file} ${config_file}
    DESTINATION ${cmake_files_install_dir})
endif()

# Where Google Test's .h files can be found.
set(gtest_build_include_dirs
  "${gtest_SOURCE_DIR}/include"
  "${gtest_SOURCE_DIR}")
include_directories(${gtest_build_include_dirs})

########################################################################
#
# Defines the gtest & gtest_main libraries.  User tests should link
# with one of them.

# Google Test libraries.  We build them using more strict warnings than what
# are used for other targets, to ensure that gtest can be compiled by a user
# aggressive about warnings.
cxx_library(gtest "${cxx_strict}" src/gtest-all.cc)
target_include_directories(gtest SYSTEM INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}/include)
