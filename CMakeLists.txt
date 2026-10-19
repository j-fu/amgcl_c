cmake_minimum_required(VERSION 3.18)
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Build type")
  message(STATUS "No build type selected, default to ${CMAKE_BUILD_TYPE}")
endif()


# Set  CMAKE_EXPORT_PACKAGE_REGISTRY  for exporting via user registry.
# See https://cmake.org/cmake/help/latest/command/export.html#exporting-packages

project(AMGCL_C C CXX)

include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
include(ExternalProject)
include(CTest)
include(CheckIncludeFile)
include(CheckTypeSize)

#
# Block sizes to be generated.
# Don't list BLOCKSIZE(1) here, this is available by default.
#
set(BLOCKSIZES "BLOCKSIZE(2) BLOCKSIZE(3) BLOCKSIZE(4) BLOCKSIZE(5) BLOCKSIZE(6) BLOCKSIZE(7) BLOCKSIZE(8)" CACHE STRING "Default block sizes to be compiled")
set(BUILD_DI_INTERFACE True CACHE BOOL "Build double - 32bit int (int) interface")
set(BUILD_DL_INTERFACE True CACHE BOOL "Build double - 64bit int (long) interface")


set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED True)

set(AMGCL_VERSION 1.4.4)
set(AMGCL_C_MAJOR_VERSION 0)
set(AMGCL_C_MINOR_VERSION 3)
set(AMGCL_C_PATCH_VERSION 0)
set(AMGCL_C_VERSION    ${AMGCL_C_MAJOR_VERSION}.${AMGCL_C_MINOR_VERSION}.${AMGCL_C_PATCH_VERSION})


find_package(Boost COMPONENTS serialization REQUIRED)

find_package(OpenMP REQUIRED)

# Thx https://stackoverflow.com/a/48460091
check_include_file(stdint.h HAVE_STDINT_H)


if (BUILD_DI_INTERFACE)
  
  check_type_size("int" SIZEOF_INT)
  if(SIZEOF_INT EQUAL "4")
    message(STATUS "sizeof(int)=4")
  else()
    message(FATAL_ERROR "Int is not 4 bytes long")
  endif()
endif()


if (BUILD_DL_INTERFACE)
  check_type_size("int64_t" SIZEOF_INT64_T)
  check_type_size("long long" SIZEOF_LONG_LONG)
  check_type_size("long" SIZEOF_LONG)
  
  if(SIZEOF_INT64_T EQUAL "8")
    set(INT64_T "int64_t")
  elseif(SIZEOF_LONG EQUAL "8")
    set(INT64_T "long")
  elseif(SIZEOF_LONG_LONG EQUAL "8")
    set(INT64_T "long long")
  else()
    message(FATAL_ERROR "Can't find suitable int64_t")
  endif()
  message(STATUS "Found int64_t: ${INT64_T}")
endif()

# This just downloads and unpacks amgcl to build/amgcl.
# No compilation here as we access it as a header only library.
set(AMGCL_DIR "${CMAKE_CURRENT_BINARY_DIR}/amgcl")

ExternalProject_Add(AMGCL
  URL https://github.com/ddemidov/amgcl/archive/refs/tags/${AMGCL_VERSION}.tar.gz
  PREFIX "${AMGCL_DIR}" #  Root directory for the external project. 
  TMP_DIR "tmp"
  STAMP_DIR "stamps" # Directory in which to store the timestamps of each step
  LOG_DIR "logs" # Directory in which to store the logs of each step.
  DOWNLOAD_DIR "downloads" # Directory in which to store downloaded files before unpacking them
  SOURCE_DIR "${AMGCL_DIR}" # Source directory into which downloaded contents will be unpacked,
  BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}" #  build directory location
  INSTALL_DIR "${AMGCL_DIR}" # Source directory into which downloaded contents will be unpacked,
  CONFIGURE_COMMAND ""
  BUILD_COMMAND ""
  INSTALL_COMMAND ""
  )

if (BUILD_DL_INTERFACE)
  set(HEADERS ${HEADERS} amgcl_c_dl.h)
  set(SOURCES ${SOURCES} amgcl_dl.cxx)
endif()

if (BUILD_DI_INTERFACE)
  set(HEADERS ${HEADERS} amgcl_c_di.h)
  set(SOURCES ${SOURCES} amgcl_di.cxx)
endif()

message(STATUS "BLOCKSIZES: ${BLOCKSIZES}")
message(STATUS "HEADERS: ${HEADERS}")
message(STATUS "SOURCES: ${SOURCES}")

# AMGCL_C code
include_directories(${AMGCL_DIR})
add_subdirectory(include)
add_subdirectory(src)
if (BUILD_DI_INTERFACE)
  add_subdirectory(examples)
endif()

#
# Export and Install
#
# From here, see https://cmake.org/cmake/help/latest/guide/importing-exporting/index.html
install(TARGETS amgcl_c
  EXPORT AMGCL_CTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )

install(FILES ${CMAKE_BINARY_DIR}/include/amgcl_c.h
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/amgcl_c
  )


install(EXPORT AMGCL_CTargets
  FILE AMGCL_CTargets.cmake
  NAMESPACE AMGCL_C::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/AMGCL_C
  )

configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/AMGCL_CConfig.cmake"
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/AMGCL_C
  )

install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/AMGCL_CConfig.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/AMGCL_CConfigVersion.cmake"
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/AMGCL_C
  )



set_property(TARGET amgcl_c PROPERTY VERSION ${AMGCL_C_VERSION})
set_property(TARGET amgcl_c PROPERTY SOVERSION ${AMGCL_C_MAJOR_VERSION})
set_property(TARGET amgcl_c PROPERTY INTERFACE_AMGCL_C_MAJOR_VERSION ${AMGCL_C_MAJOR_VERSION})
set_property(TARGET amgcl_c APPEND PROPERTY COMPATIBLE_INTERFACE_STRING AMGCL_C_MAJOR_VERSION)

# generate the version file for the config file
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/AMGCL_CConfigVersion.cmake"
  VERSION "${AMGCL_C_VERSION}"
  COMPATIBILITY AnyNewerVersion
  )

export(EXPORT AMGCL_CTargets
       FILE "${CMAKE_CURRENT_BINARY_DIR}/AMGCL_CTargets.cmake"
       NAMESPACE AMGCL_C::
       )

# Register this build tree in the CMake user package registry
# (~/.cmake/packages/AMGCL_C) so that downstream projects (e.g. pdelib with
# -DWITH_AMGCL_C=ON) find a freshly built AMGCL_C without any
# -DCMAKE_PREFIX_PATH. export(PACKAGE) does nothing unless
# CMAKE_EXPORT_PACKAGE_REGISTRY is enabled, so enable it here by default;
# an explicit -DCMAKE_EXPORT_PACKAGE_REGISTRY=OFF switches it back off.
# An installation discovered via CMAKE_PREFIX_PATH or a default install
# prefix still takes precedence (the registry is searched last); stale
# registry entries are ignored by find_package.
if(NOT DEFINED CMAKE_EXPORT_PACKAGE_REGISTRY)
  set(CMAKE_EXPORT_PACKAGE_REGISTRY TRUE)
endif()
export(PACKAGE AMGCL_C)
