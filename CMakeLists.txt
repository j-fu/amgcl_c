cmake_minimum_required(VERSION 3.14)
if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING "Build type")
    message(STATUS "No build type selected, default to ${CMAKE_BUILD_TYPE}")
endif()

project(AMGCL-C C CXX)


find_package(Boost COMPONENTS
    program_options
    serialization
    unit_test_framework
    )

include(ExternalProject)
set(AMGCL_DIR "${CMAKE_CURRENT_BINARY_DIR}/amgcl")
ExternalProject_Add(AMGCL
  URL https://github.com/ddemidov/amgcl/archive/refs/tags/1.4.4.tar.gz
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


add_subdirectory(src)
include_directories(include)
add_subdirectory(examples)

set(CONF_INCLUDE_DIRS "${PROJECT_SOURCE_DIR}/include")