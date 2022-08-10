cmake_minimum_required(VERSION 3.13)

project(gRPCexample LANGUAGES CXX)

set(CMAKE_BUILD_TYPE Debug)

include(FetchContent)
FetchContent_Declare(
  gRPC
  GIT_REPOSITORY https://github.com/grpc/grpc
  GIT_TAG        v1.48.0  # e.g v1.28.0
)
set(FETCHCONTENT_QUIET OFF)
FetchContent_MakeAvailable(gRPC)

include(FindProtobuf)
set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(Protobuf REQUIRED)
include_directories(${CMAKE_BINARY_DIR})

set(_PROTOBUF_LIBPROTOBUF protobuf::libprotobuf)
set(_REFLECTION grpc++_reflection)
set(_PROTOBUF_PROTOC $<TARGET_FILE:protobuf::protoc>)
set(_GRPC_GRPCPP grpc++)
set(_GRPC_CPP_PLUGIN_EXECUTABLE "${CMAKE_CURRENT_BINARY_DIR}/_deps/grpc-build/grpc_cpp_plugin")

get_filename_component(sample_proto "src/proto/sample.proto" ABSOLUTE)
get_filename_component(sample_proto_path "${sample_proto}" PATH)

set(sample_proto_sources "${CMAKE_CURRENT_BINARY_DIR}/sample.pb.cc")
set(sample_proto_headers "${CMAKE_CURRENT_BINARY_DIR}/sample.pb.h")
set(sample_grpc_sources "${CMAKE_CURRENT_BINARY_DIR}/sample.grpc.pb.cc")
set(sample_grpc_headers "${CMAKE_CURRENT_BINARY_DIR}/sample.grpc.pb.h")

add_custom_command(
        OUTPUT "${sample_proto_sources}" "${sample_proto_headers}" "${sample_grpc_sources}" "${sample_grpc_headers}"
        COMMAND ${_PROTOBUF_PROTOC}
        ARGS --grpc_out "${CMAKE_CURRENT_BINARY_DIR}" --cpp_out "${CMAKE_CURRENT_BINARY_DIR}"
        -I "${sample_proto_path}"
        --plugin=protoc-gen-grpc="${_GRPC_CPP_PLUGIN_EXECUTABLE}"
        "${sample_proto}"
        DEPENDS "${sample_proto}")

include_directories("${CMAKE_CURRENT_BINARY_DIR}")

add_library(sample_grpc_proto
        ${sample_proto_sources}
        ${sample_proto_headers}
        ${sample_grpc_sources}
        ${sample_grpc_headers})

target_link_libraries(sample_grpc_proto
        ${_PROTOBUF_LIBPROTOBUF}
        ${_REFLECTION}
        ${_GRPC_GRPCPP})

add_executable(${PROJECT_NAME} "src/main.cpp")
target_link_libraries(${PROJECT_NAME}
        sample_grpc_proto
        ${_PROTOBUF_LIBPROTOBUF}
        ${_REFLECTION}
        ${_GRPC_GRPCPP})


add_executable(${PROJECT_NAME}_client "src/client.cpp")
target_link_libraries(${PROJECT_NAME}_client
        sample_grpc_proto
        ${_PROTOBUF_LIBPROTOBUF}
        ${_REFLECTION}
        ${_GRPC_GRPCPP})