# This file was generated from BUILD using tools/make_cmakelists.py.

cmake_minimum_required(VERSION 3.10)

if(${CMAKE_VERSION} VERSION_LESS 3.12)
    cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
else()
    cmake_policy(VERSION 3.12)
endif()

cmake_policy(SET CMP0048 NEW)

project(ml_metadata)


# Prevent CMake from setting -rdynamic on Linux (!!).
SET(CMAKE_SHARED_LIBRARY_LINK_C_FLAGS "")
SET(CMAKE_SHARED_LIBRARY_LINK_CXX_FLAGS "")

# Set default build type.
if(NOT CMAKE_BUILD_TYPE)
  message(STATUS "Setting build type to 'RelWithDebInfo' as none was specified.")
  set(CMAKE_BUILD_TYPE "RelWithDebInfo" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
endif()

# When using Ninja, compiler output won't be colorized without this.
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG(-fdiagnostics-color=always SUPPORTS_COLOR_ALWAYS)
if(SUPPORTS_COLOR_ALWAYS)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fdiagnostics-color=always")
endif()

# Implement ASAN/UBSAN options
if(UPB_ENABLE_ASAN)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=address")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -fsanitize=address")
endif()

if(UPB_ENABLE_UBSAN)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=undefined")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=address")
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -fsanitize=address")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -fsanitize=address")
endif()

include_directories(.)
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${Protobuf_INCLUDE_DIRS})

if(APPLE)
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -undefined dynamic_lookup -flat_namespace")
elseif(UNIX)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--build-id")
endif()

enable_testing()

# START PROTO
find_package(Protobuf REQUIRED)

SET(srcs
  "metadata_source.proto")
# Generate C++ files from protobuf definitions
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS 
  ${srcs})

# Create a library from the generated protobuf files
add_library(metadata_source_proto
    ${PROTO_SRCS}
    ${PROTO_HDRS})
target_link_libraries(metadata_source_proto
  Protobuf_LIBRARIES)
target_include_directories(metadata_source_proto
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(metadata_source_proto PROPERTIES
  CXX_VISIBILITY_PRESET default)

SET(srcs
  "metadata_store.proto")
# Generate C++ files from protobuf definitions
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS 
  ${srcs})

# Create a library from the generated protobuf files
add_library(metadata_store_proto
    ${PROTO_SRCS}
    ${PROTO_HDRS})
target_link_libraries(metadata_store_proto
  Protobuf_LIBRARIES)
target_include_directories(metadata_store_proto
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(metadata_store_proto PROPERTIES
  CXX_VISIBILITY_PRESET default)

SET(srcs
  "metadata_store_service.proto")
# Generate C++ files from protobuf definitions
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS 
  ${srcs})

# Create a library from the generated protobuf files
add_library(metadata_store_service_proto
    ${PROTO_SRCS}
    ${PROTO_HDRS})
target_link_libraries(metadata_store_service_proto
  Protobuf_LIBRARIES)
target_include_directories(metadata_store_service_proto
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
set_target_properties(metadata_store_service_proto PROPERTIES
  CXX_VISIBILITY_PRESET default)