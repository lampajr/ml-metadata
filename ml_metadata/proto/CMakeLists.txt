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

# include_directories(.)
# include_directories(${CMAKE_CURRENT_BINARY_DIR})
# include_directories(${Protobuf_INCLUDE_DIRS})
include_directories("/home/alampare/tools/grpc/include")

if(APPLE)
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -undefined dynamic_lookup -flat_namespace")
elseif(UNIX)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--build-id")
endif()

enable_testing()

find_package(Protobuf REQUIRED)
find_package(gRPC CONFIG REQUIRED)
find_program(GRPC_CPP_PLUGIN grpc_cpp_plugin) # Get full path to plugin

function(PROTOBUF_GENERATE_GRPC_CPP SRCS HDRS)
  if(NOT ARGN)
    message(SEND_ERROR "Error: PROTOBUF_GENERATE_GRPC_CPP() called without any proto files")
    return()
  endif()

  if(PROTOBUF_GENERATE_CPP_APPEND_PATH) # This variable is common for all types of output.
    # Create an include path for each file specified
    foreach(FIL ${ARGN})
      get_filename_component(ABS_FIL ${FIL} ABSOLUTE)
      get_filename_component(ABS_PATH ${ABS_FIL} PATH)
      list(FIND _protobuf_include_path ${ABS_PATH} _contains_already)
      if(${_contains_already} EQUAL -1)
          list(APPEND _protobuf_include_path -I ${ABS_PATH})
      endif()
    endforeach()
  else()
    set(_protobuf_include_path -I ${CMAKE_CURRENT_SOURCE_DIR})
  endif()

  if(DEFINED PROTOBUF_IMPORT_DIRS)
    foreach(DIR ${Protobuf_IMPORT_DIRS})
      get_filename_component(ABS_PATH ${DIR} ABSOLUTE)
      list(FIND _protobuf_include_path ${ABS_PATH} _contains_already)
      if(${_contains_already} EQUAL -1)
          list(APPEND _protobuf_include_path -I ${ABS_PATH})
      endif()
    endforeach()
  endif()

  set(${SRCS})
  set(${HDRS})
  foreach(FIL ${ARGN})
    get_filename_component(ABS_FIL ${FIL} ABSOLUTE)
    get_filename_component(FIL_WE ${FIL} NAME_WE)

    list(APPEND ${SRCS} "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.grpc.pb.cc")
    list(APPEND ${HDRS} "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.grpc.pb.h")

    add_custom_command(
      OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.grpc.pb.cc"
             "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.grpc.pb.h"
      COMMAND  ${Protobuf_PROTOC_EXECUTABLE}
      ARGS --grpc_out=${CMAKE_CURRENT_BINARY_DIR}
           --plugin=protoc-gen-grpc=${GRPC_CPP_PLUGIN}
           ${_protobuf_include_path} ${ABS_FIL}
      DEPENDS ${ABS_FIL} ${Protobuf_PROTOC_EXECUTABLE}
      COMMENT "Running gRPC C++ protocol buffer compiler on ${FIL}"
      VERBATIM)
  endforeach()

  set_source_files_properties(${${SRCS}} ${${HDRS}} PROPERTIES GENERATED TRUE)
  set(${SRCS} ${${SRCS}} PARENT_SCOPE)
  set(${HDRS} ${${HDRS}} PARENT_SCOPE)
endfunction()

# function(PROTOBUF_GENERATE_CPP_CUSTOM SRCS HDRS)
#   if(NOT ARGN)
#     message(SEND_ERROR "Error: PROTOBUF_GENERATE_CPP_CUSTOM() called without any proto files")
#     return()
#   endif()

#   if(PROTOBUF_GENERATE_CPP_APPEND_PATH) # This variable is common for all types of output.
#     # Create an include path for each file specified
#     foreach(FIL ${ARGN})
#       get_filename_component(ABS_FIL ${FIL} ABSOLUTE)
#       get_filename_component(ABS_PATH ${ABS_FIL} PATH)
#       list(FIND _protobuf_include_path ${ABS_PATH} _contains_already)
#       if(${_contains_already} EQUAL -1)
#           list(APPEND _protobuf_include_path -I ${ABS_PATH})
#       endif()
#     endforeach()
#   else()
#     set(_protobuf_include_path -I ${CMAKE_CURRENT_SOURCE_DIR})
#   endif()

#   if(DEFINED PROTOBUF_IMPORT_DIRS)
#     foreach(DIR ${Protobuf_IMPORT_DIRS})
#       get_filename_component(ABS_PATH ${DIR} ABSOLUTE)
#       list(FIND _protobuf_include_path ${ABS_PATH} _contains_already)
#       if(${_contains_already} EQUAL -1)
#           list(APPEND _protobuf_include_path -I ${ABS_PATH})
#       endif()
#     endforeach()
#   endif()

#   set(${SRCS})
#   set(${HDRS})
#   foreach(FIL ${ARGN})
#     get_filename_component(ABS_FIL ${FIL} ABSOLUTE)
#     get_filename_component(FIL_WE ${FIL} NAME_WE)

#     list(APPEND ${SRCS} "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.cc")
#     list(APPEND ${HDRS} "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.h")

#     add_custom_command(
#       OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.cc"
#              "${CMAKE_CURRENT_BINARY_DIR}/${FIL_WE}.pb.h"
#       COMMAND  ${Protobuf_PROTOC_EXECUTABLE}
#       ARGS --cpp_out=${CMAKE_CURRENT_BINARY_DIR}
#            ${_protobuf_include_path} ${ABS_FIL}
#       DEPENDS ${ABS_FIL} ${Protobuf_PROTOC_EXECUTABLE}
#       COMMENT "Running C++ protocol buffer compiler on ${FIL}"
#       VERBATIM)
#   endforeach()

#   set_source_files_properties(${${SRCS}} ${${HDRS}} PROPERTIES GENERATED TRUE)
#   set(${SRCS} ${${SRCS}} PARENT_SCOPE)
#   set(${HDRS} ${${HDRS}} PARENT_SCOPE)
# endfunction()

# START PROTO

set(Protobuf_IMPORT_DIRS ${CMAKE_CURRENT_SOURCE_DIR})
set(PROTOBUF_IMPORT_DIRS ${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${Protobuf_IMPORT_DIRS})


# Generate C++ files from protobuf definitions
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS 
  metadata_source.proto)

# Create a library from the generated protobuf files
add_library(metadata_source_proto
    ${PROTO_SRCS}
    ${PROTO_HDRS})
target_link_libraries(metadata_source_proto
  protobuf::libprotobuf)
target_include_directories(metadata_source_proto
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

# Generate C++ files from protobuf definitions
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS 
  metadata_store.proto)

# Create a library from the generated protobuf files
add_library(metadata_store_proto
    ${PROTO_SRCS}
    ${PROTO_HDRS})
target_link_libraries(metadata_store_proto
  protobuf::libprotobuf)
target_include_directories(metadata_store_proto
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})


# Generate C++ files from protobuf definitions
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS 
  metadata_store_service.proto)

# Create a library from the generated protobuf files
add_library(metadata_store_service_proto
    ${PROTO_SRCS}
    ${PROTO_HDRS})
target_link_libraries(metadata_store_service_proto
  PUBLIC 
    metadata_store_proto
    protobuf::libprotobuf)
target_include_directories(metadata_store_service_proto
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

# Generate C++ files from protobuf definitions
PROTOBUF_GENERATE_GRPC_CPP(PROTO_SRCS PROTO_HDRS 
  metadata_store_service.proto)

# Create a library from the generated protobuf files
add_library(metadata_store_service_grpc_proto
    ${PROTO_SRCS}
    ${PROTO_HDRS})
target_link_libraries(metadata_store_service_grpc_proto
  PUBLIC 
    metadata_store_proto
    protobuf::libprotobuf)
target_include_directories(metadata_store_service_grpc_proto
  PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

