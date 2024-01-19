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

if(APPLE)
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -undefined dynamic_lookup -flat_namespace")
elseif(UNIX)
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--build-id")
endif()

enable_testing()

include_directories(.)
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(${Protobuf_INCLUDE_DIRS})
include_directories("/usr/include/mysql")
include_directories("/usr/include/glog")
include_directories("/usr/include/absl")
include_directories("/home/alampare/tools/grpc/include")

find_package(Protobuf REQUIRED)
find_package(gRPC REQUIRED)
find_package(gflags REQUIRED)
find_package(absl REQUIRED)

set(Protobuf_IMPORT_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/ml_metadata/proto)
include_directories(${Protobuf_IMPORT_DIRS})

SET(PROTO_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/ml_metadata/proto")
add_subdirectory(${PROTO_DIRECTORY} ${PROJECT_SOURCE_DIR}/proto_gen)
include_directories(${PROJECT_SOURCE_DIR}/proto_gen)

SET(PROTO_TESTING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/ml_metadata/proto/testing")
add_subdirectory(${PROTO_TESTING_DIRECTORY} ${PROJECT_SOURCE_DIR}/proto_testing_gen)
include_directories(${PROJECT_SOURCE_DIR}/proto_gen)

SET(SIMPLE_TYPES_PROTO_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/ml_metadata/simple_types/proto")
add_subdirectory(${SIMPLE_TYPES_PROTO_DIRECTORY} ${PROJECT_SOURCE_DIR}/simple_types_proto_gen)
include_directories(${PROJECT_SOURCE_DIR}/simple_types_proto_gen)

add_library(simple_types_constants
  ml_metadata/simple_types/simple_types_constants.cc
  ml_metadata/simple_types/simple_types_constants.h)
target_link_libraries(simple_types_constants
  absl::core_headers
  absl::strings)
set_target_properties(simple_types_constants PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_source
  ml_metadata/metadata_store/metadata_source.cc
  ml_metadata/metadata_store/metadata_source.h)
target_link_libraries(metadata_source
  types
  absl::status
  absl::statusor
  absl::strings
  return_utils)
set_target_properties(metadata_source PROPERTIES LINKER_LANGUAGE CXX)
target_include_directories(metadata_source PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})

add_library(metadata_access_object_base
  ml_metadata/metadata_store/metadata_access_object.h)
target_link_libraries(metadata_access_object_base
  constants
  metadata_source
  absl::flat_hash_map
  absl::node_hash_map
  absl::status
  absl::strings
  absl::time
  absl::optional
  absl::span
  metadata_source_proto
  metadata_store_proto
  protobuf::libprotobuf)
set_target_properties(metadata_access_object_base PROPERTIES LINKER_LANGUAGE CXX)

add_library(constants
  ml_metadata/metadata_store/constants.h)
target_link_libraries(constants
  absl::strings)
set_target_properties(constants PROPERTIES LINKER_LANGUAGE CXX)

add_library(record_parsing_utils
  ml_metadata/metadata_store/record_parsing_utils.cc
  ml_metadata/metadata_store/record_parsing_utils.h)
target_link_libraries(record_parsing_utils
  constants
  absl::flat_hash_set
  absl::status
  absl::strings
  metadata_source_proto
  metadata_store_proto
  return_utils
  glog)
set_target_properties(record_parsing_utils PROPERTIES LINKER_LANGUAGE CXX)

add_library(rdbms_metadata_access_object
  ml_metadata/metadata_store/rdbms_metadata_access_object.cc
  ml_metadata/metadata_store/rdbms_metadata_access_object.h)
target_link_libraries(rdbms_metadata_access_object
  constants
  list_operation_util
  metadata_access_object_base
  metadata_source
  query_executor
  record_parsing_utils
  protobuf::libprotobuf
  absl::algorithm_container
  absl::flat_hash_map
  absl::flat_hash_set
  absl::node_hash_map
  absl::status
  absl::statusor
  absl::strings
  absl::time
  absl::optional
  absl::span
  metadata_source_proto
  metadata_store_proto
  simple_types_constants
  field_mask_utils
  return_utils
  struct_utils
  glog
  # com_google_protobuf//:cc_wkt_protos
  protobuf::libprotobuf)
set_target_properties(rdbms_metadata_access_object PROPERTIES LINKER_LANGUAGE CXX)

add_library(rdbms_metadata_access_object_test
  ml_metadata/metadata_store/rdbms_metadata_access_object_test.cc
  ml_metadata/metadata_store/rdbms_metadata_access_object_test.h)
target_link_libraries(rdbms_metadata_access_object_test
  rdbms_metadata_access_object
  test_util
  types
  com_google_googletest//:gtest
  absl::flat_hash_map
  absl::status
  absl::strings
  absl::time
  metadata_source_proto
  metadata_store_proto
  return_utils
  glog)
set_target_properties(rdbms_metadata_access_object_test PROPERTIES LINKER_LANGUAGE CXX)

add_library(query_executor
  ml_metadata/metadata_store/query_executor.cc
  ml_metadata/metadata_store/query_executor.h)
target_link_libraries(query_executor
  constants
  metadata_source
  absl::status
  absl::strings
  absl::time
  absl::optional
  absl::span
  metadata_source_proto
  metadata_store_proto
  return_utils
  glog)
set_target_properties(query_executor PROPERTIES LINKER_LANGUAGE CXX)

add_library(query_executor_test
  ml_metadata/metadata_store/query_executor_test.cc
  ml_metadata/metadata_store/query_executor_test.h)
target_link_libraries(query_executor_test
  constants
  metadata_source
  query_executor
  test_util
  com_google_googletest//:gtest
  absl::status
  absl::strings
  absl::time
  absl::optional
  absl::span
  metadata_source_proto
  metadata_store_proto
  return_utils
  glog)
set_target_properties(query_executor_test PROPERTIES LINKER_LANGUAGE CXX)

add_library(mysql_query_config_executor_test
  ml_metadata/metadata_store/mysql_query_config_executor_test.cc)
target_link_libraries(mysql_query_config_executor_test
  metadata_source
  mysql_metadata_source
  # query_config_executor # NOT BUILT DUE TO MISSING ZETASQL
  query_executor
  query_executor_test
  test_mysql_metadata_source_initializer
  com_google_googletest//:gtest_main
  absl::memory
  metadata_source_proto
  metadata_source_query_config
  glog)
set_target_properties(mysql_query_config_executor_test PROPERTIES LINKER_LANGUAGE CXX)

# NOT BUILT DUE TO MISSING ZETASQL
# add_library(query_config_executor
#   ml_metadata/metadata_store/query_config_executor.cc
#   ml_metadata/metadata_store/query_config_executor.h)
# target_link_libraries(query_config_executor
#   list_operation_query_helper
#   metadata_source
#   query_executor
#   glog
#  # com_google_protobuf//:cc_wkt_protos
#   protobuf::libprotobuf
#   absl::strings
#   absl::status
#   absl::statusor
#   /ml_metadata/query:filter_query_ast_resolver
#   /ml_metadata/query:filter_query_builder
#   metadata_source_proto
#   metadata_store_proto
#   return_utils
#   struct_utils)
# set_target_properties(query_config_executor PROPERTIES LINKER_LANGUAGE CXX)

add_library(list_operation_query_helper
  ml_metadata/metadata_store/list_operation_query_helper.cc
  ml_metadata/metadata_store/list_operation_query_helper.h)
target_link_libraries(list_operation_query_helper
  constants
  list_operation_util
  types
  absl::status
  absl::strings
  absl::optional
  absl::span
  metadata_store_proto
  return_utils)
set_target_properties(list_operation_query_helper PROPERTIES LINKER_LANGUAGE CXX)

add_library(list_operation_util
  ml_metadata/metadata_store/list_operation_util.cc
  ml_metadata/metadata_store/list_operation_util.h)
target_link_libraries(list_operation_util
  types
  absl::status
  absl::strings
  absl::span
  metadata_store_proto)
set_target_properties(list_operation_util PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_access_object_factory
  ml_metadata/metadata_store/metadata_access_object_factory.cc
  ml_metadata/metadata_store/metadata_access_object.h
  ml_metadata/metadata_store/metadata_access_object_factory.h)
target_link_libraries(metadata_access_object_factory
  constants
  metadata_access_object_base
  metadata_source
  # postgresql_query_executor # NOT BUILT DUE TO MISSING ZETASQL
  # query_config_executor # NOT BUILT DUE TO MISSING ZETASQL
  rdbms_metadata_access_object
  absl::flat_hash_map
  absl::node_hash_map
  absl::memory
  absl::status
  absl::strings
  absl::time
  absl::optional
  absl::span
  metadata_source_proto
  metadata_store_proto
  return_utils
  protobuf::libprotobuf)
set_target_properties(metadata_access_object_factory PROPERTIES LINKER_LANGUAGE CXX)

add_library(transaction_executor
  ml_metadata/metadata_store/transaction_executor.cc
  ml_metadata/metadata_store/transaction_executor.h)
target_link_libraries(transaction_executor
  metadata_source
  absl::status
  metadata_store_proto
  return_utils)
set_target_properties(transaction_executor PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_store_service_interface
  ml_metadata/metadata_store/metadata_store_service_interface.h)
target_link_libraries(metadata_store_service_interface
  absl::status
  metadata_store_service_proto)
set_target_properties(metadata_store_service_interface PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_store
  ml_metadata/metadata_store/metadata_store.cc
  ml_metadata/metadata_store/metadata_store.h)
target_link_libraries(metadata_store
  constants
  metadata_access_object_factory
  metadata_source
  metadata_store_service_interface
  rdbms_metadata_access_object
  simple_types_util
  transaction_executor
  protobuf::libprotobuf
  absl::algorithm_container
  absl::flat_hash_map
  absl::flat_hash_set
  absl::node_hash_map
  absl::memory
  absl::status
  absl::strings
  absl::time
  absl::optional
  absl::span
  metadata_store_proto
  metadata_store_service_proto
  simple_types_constants
  simple_types_proto
  return_utils
  glog)
set_target_properties(metadata_store PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_store_test_suite
  ml_metadata/metadata_store/metadata_store_test_suite.cc
  ml_metadata/metadata_store/metadata_store_test_suite.h)
target_link_libraries(metadata_store_test_suite
  metadata_store
  simple_types_util
  test_util
  com_google_googletest//:gtest
  absl::algorithm_container
  absl::status
  absl::strings
  absl::str_format
  metadata_store_proto
  metadata_store_service_proto
  simple_types_proto
  glog)
set_target_properties(metadata_store_test_suite PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_store_headers
  ml_metadata/metadata_store/metadata_access_object.h
  ml_metadata/metadata_store/metadata_access_object_factory.h
  ml_metadata/metadata_store/metadata_source.h
  ml_metadata/metadata_store/metadata_store.h
  ml_metadata/metadata_store/metadata_store_factory.h
  ml_metadata/metadata_store/metadata_store_service_interface.h
  ml_metadata/metadata_store/transaction_executor.h)
target_link_libraries(metadata_store_headers
  constants
  types
  absl::flat_hash_map
  absl::node_hash_map
  absl::status
  absl::statusor
  absl::strings
  absl::time
  absl::optional
  absl::span
  metadata_source_proto
  metadata_store_proto
  metadata_store_service_proto
  protobuf::libprotobuf)
set_target_properties(metadata_store_headers PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_store_factory
  ml_metadata/metadata_store/metadata_store_factory.cc
  ml_metadata/metadata_store/metadata_store_factory.h)
target_link_libraries(metadata_store_factory
  metadata_store
  mysql_metadata_source
  postgresql_metadata_source
  sqlite_metadata_source
  transaction_executor
  absl::memory
  absl::status
  metadata_store_proto
  metadata_source_query_config
  return_utils)
set_target_properties(metadata_store_factory PROPERTIES LINKER_LANGUAGE CXX)

add_library(sqlite_metadata_source_util
  ml_metadata/metadata_store/sqlite_metadata_source_util.cc
  ml_metadata/metadata_store/sqlite_metadata_source_util.h)
target_link_libraries(sqlite_metadata_source_util
  constants
  absl::statusor
  absl::strings
  metadata_source_proto
  sqlite3)
set_target_properties(sqlite_metadata_source_util PROPERTIES LINKER_LANGUAGE CXX)

add_library(sqlite_metadata_source
  ml_metadata/metadata_store/sqlite_metadata_source.cc
  ml_metadata/metadata_store/sqlite_metadata_source.h)
target_link_libraries(sqlite_metadata_source
  metadata_source
  sqlite_metadata_source_util
  absl::status
  absl::statusor
  absl::strings
  absl::time
  metadata_store_proto
  glog
  sqlite3)
set_target_properties(sqlite_metadata_source PROPERTIES LINKER_LANGUAGE CXX)

add_library(test_util
  ml_metadata/metadata_store/test_util.h)
target_link_libraries(test_util
  protobuf::libprotobuf
  com_google_googletest//:gtest
  glog)
set_target_properties(test_util PROPERTIES LINKER_LANGUAGE CXX)

add_library(simple_types_util
  ml_metadata/metadata_store/simple_types_util.cc
  ml_metadata/metadata_store/simple_types_util.h)
target_link_libraries(simple_types_util
  protobuf::libprotobuf
  absl::flat_hash_map
  absl::status
  absl::strings
  metadata_store_proto
  simple_types_constants
  simple_types_proto)
set_target_properties(simple_types_util PROPERTIES LINKER_LANGUAGE CXX)

add_library(types
  ml_metadata/metadata_store/types.h)
set_target_properties(types PROPERTIES LINKER_LANGUAGE CXX)

add_library(mysql_metadata_source
  ml_metadata/metadata_store/mysql_metadata_source.cc
  ml_metadata/metadata_store/mysql_metadata_source.h)
target_link_libraries(mysql_metadata_source
  constants
  metadata_source
  sqlite_metadata_source_util
  types
  absl::status
  absl::statusor
  absl::strings
  metadata_source_proto
  metadata_store_proto
  return_utils
  glog
  mysqlclient)
set_target_properties(mysql_metadata_source PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_store_service_impl
  ml_metadata/metadata_store/metadata_store_service_impl.cc
  ml_metadata/metadata_store/metadata_store_service_impl.h)
target_link_libraries(metadata_store_service_impl
  metadata_store
  metadata_store_factory
  absl::status
  metadata_store_service_grpc_proto
  gRPC::grpc++
  glog)
set_target_properties(metadata_store_service_impl PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_access_object_test
  ml_metadata/metadata_store/metadata_access_object_test.cc
  ml_metadata/metadata_store/metadata_access_object_test.h)
target_link_libraries(metadata_access_object_test
  metadata_access_object_base
  metadata_source
  test_util
  types
  com_google_googletest//:gtest
  absl::flat_hash_map
  absl::node_hash_map
  absl::status
  absl::statusor
  absl::strings
  absl::str_format
  absl::time
  absl::optional
  absl::span
  metadata_source_proto
  metadata_store_proto
  /ml_metadata/proto/testing:mock_proto
  return_utils
  gflags_nothreads
  glog
  # com_google_protobuf//:cc_wkt_protos
  protobuf::libprotobuf)
set_target_properties(metadata_access_object_test PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_source_test_suite
  ml_metadata/metadata_store/metadata_source_test_suite.cc
  ml_metadata/metadata_store/metadata_source_test_suite.h)
target_link_libraries(metadata_source_test_suite
  constants
  metadata_source
  test_util
  com_google_googletest//:gtest
  absl::status
  absl::strings
  metadata_source_proto)
set_target_properties(metadata_source_test_suite PROPERTIES LINKER_LANGUAGE CXX)

add_library(test_mysql_metadata_source_initializer
  ml_metadata/metadata_store/test_mysql_metadata_source_initializer.h)
target_link_libraries(test_mysql_metadata_source_initializer
  mysql_metadata_source)
set_target_properties(test_mysql_metadata_source_initializer PROPERTIES LINKER_LANGUAGE CXX)

add_library(test_standalone_mysql_metadata_source_initializer
  ml_metadata/metadata_store/test_standalone_mysql_metadata_source_initializer.cc)
target_link_libraries(test_standalone_mysql_metadata_source_initializer
  mysql_metadata_source
  test_mysql_metadata_source_initializer
  absl::memory
  metadata_store_proto
  gflags_nothreads
  glog)
set_target_properties(test_standalone_mysql_metadata_source_initializer PROPERTIES LINKER_LANGUAGE CXX)

add_library(mysql_metadata_source_test
  ml_metadata/metadata_store/mysql_metadata_source_test.cc)
target_link_libraries(mysql_metadata_source_test
  metadata_source
  metadata_source_test_suite
  mysql_metadata_source
  test_mysql_metadata_source_initializer
  com_google_googletest//:gtest
  com_google_googletest//:gtest_main
  absl::memory
  gflags_nothreads
  glog)
set_target_properties(mysql_metadata_source_test PROPERTIES LINKER_LANGUAGE CXX)

add_library(mysql_metadata_access_object_test
  ml_metadata/metadata_store/mysql_metadata_access_object_test.cc)
target_link_libraries(mysql_metadata_access_object_test
  metadata_access_object_factory
  metadata_access_object_test
  metadata_source
  mysql_metadata_source
  test_mysql_metadata_source_initializer
  com_google_googletest//:gtest
  absl::memory
  absl::status
  metadata_source_proto
  metadata_source_query_config
  glog)
set_target_properties(mysql_metadata_access_object_test PROPERTIES LINKER_LANGUAGE CXX)

add_library(postgresql_metadata_source_test
  ml_metadata/metadata_store/postgresql_metadata_source_test.cc)
target_link_libraries(postgresql_metadata_source_test
  metadata_source
  metadata_source_test_suite
  postgresql_metadata_source
  test_postgresql_metadata_source_initializer
  com_google_googletest//:gtest
  com_google_googletest//:gtest_main
  com_google_absl//absl/flags:flag
  absl::memory
  gflags_nothreads
  glog)
set_target_properties(postgresql_metadata_source_test PROPERTIES LINKER_LANGUAGE CXX)

add_library(test_postgresql_standalone_metadata_source_initializer
  ml_metadata/metadata_store/test_postgresql_standalone_metadata_source_initializer.cc)
target_link_libraries(test_postgresql_standalone_metadata_source_initializer
  postgresql_metadata_source
  test_postgresql_metadata_source_initializer
  absl::memory
  metadata_store_proto
  gflags_nothreads
  glog
  pq)
set_target_properties(test_postgresql_standalone_metadata_source_initializer PROPERTIES LINKER_LANGUAGE CXX)

add_library(postgresql_metadata_source
  ml_metadata/metadata_store/postgresql_metadata_source.cc
  ml_metadata/metadata_store/postgresql_metadata_source.h)
target_link_libraries(postgresql_metadata_source
  constants
  metadata_source
  sqlite_metadata_source_util
  types
  absl::status
  absl::statusor
  absl::strings
  metadata_source_proto
  metadata_store_proto
  return_utils
  glog
  pq)
set_target_properties(postgresql_metadata_source PROPERTIES LINKER_LANGUAGE CXX)

# NOT BUILT DUE TO MISSING ZETASQL?
# add_library(postgresql_query_executor
#   ml_metadata/metadata_store/postgresql_query_executor.cc
#   ml_metadata/metadata_store/postgresql_query_executor.h)
# target_link_libraries(postgresql_query_executor
#   list_operation_query_helper
#   metadata_source
#   query_config_executor # NOT BUILT DUE TO MISSING ZETASQL
#   query_executor
#   glog
# #  com_google_protobuf//:cc_wkt_protos
#   protobuf::libprotobuf
#   absl::status
#   absl::statusor
#   absl::strings
#   metadata_source_proto
#   metadata_store_proto
#   /ml_metadata/query:filter_query_ast_resolver
#   /ml_metadata/query:filter_query_builder
#   return_utils
#   struct_utils)
# set_target_properties(postgresql_query_executor PROPERTIES LINKER_LANGUAGE CXX)

# add_library(postgresql_query_executor_test
#   ml_metadata/metadata_store/postgresql_query_executor_test.cc)
# target_link_libraries(postgresql_query_executor_test
#   postgresql_query_executor
#   query_executor
#   query_executor_test
#   test_postgresql_metadata_source_initializer
#   com_google_googletest//:gtest_main
#   absl::memory
#   metadata_source_query_config)
# set_target_properties(postgresql_query_executor_test PROPERTIES LINKER_LANGUAGE CXX)

add_library(test_postgresql_metadata_source_initializer
  ml_metadata/metadata_store/test_postgresql_metadata_source_initializer.h)
target_link_libraries(test_postgresql_metadata_source_initializer
  postgresql_metadata_source)
set_target_properties(test_postgresql_metadata_source_initializer PROPERTIES LINKER_LANGUAGE CXX)

add_library(metadata_source_query_config
  ml_metadata/util/metadata_source_query_config.cc
  ml_metadata/util/metadata_source_query_config.h)
target_link_libraries(metadata_source_query_config
  protobuf::libprotobuf
  absl::strings
  metadata_source_proto
  glog)
set_target_properties(metadata_source_query_config PROPERTIES LINKER_LANGUAGE CXX)

add_library(return_utils
  ml_metadata/util/return_utils.h)
set_target_properties(return_utils PROPERTIES LINKER_LANGUAGE CXX)

add_library(struct_utils
  ml_metadata/util/struct_utils.cc
  ml_metadata/util/struct_utils.h)
target_link_libraries(struct_utils
  protobuf::libprotobuf
  absl::status
  absl::strings
  # com_google_protobuf//:cc_wkt_protos
  protobuf::libprotobuf)
set_target_properties(struct_utils PROPERTIES LINKER_LANGUAGE CXX)

add_library(field_mask_utils
  ml_metadata/util/field_mask_utils.cc
  ml_metadata/util/field_mask_utils.h)
target_link_libraries(field_mask_utils
  absl::flat_hash_set
  absl::status
  absl::statusor
  absl::strings
  metadata_store_proto
  protobuf::libprotobuf)
set_target_properties(field_mask_utils PROPERTIES LINKER_LANGUAGE CXX)

add_executable(metadata_store_server
  ml_metadata/metadata_store/metadata_store_server_main.cc)
target_link_libraries(metadata_store_server
  metadata_store
  metadata_store_factory
  metadata_store_service_impl
  protobuf::libprotobuf
  absl::status
  absl::statusor
  absl::strings
  metadata_store_proto
  gflags_nothreads
  gRPC::grpc++
  glog)