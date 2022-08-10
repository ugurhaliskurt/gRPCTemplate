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
find_package(Protobuf REQUIRED)
include_directories(${CMAKE_BINARY_DIR})

file(GLOB ProtoFiles "${CMAKE_CURRENT_SOURCE_DIR}/src/proto/*.proto")
PROTOBUF_GENERATE_CPP(ProtoSources ProtoHeaders ${ProtoFiles})
add_library(messages STATIC ${ProtoSources} ${ProtoHeaders})

add_executable(${PROJECT_NAME} "src/main.cpp")
target_link_libraries(${PROJECT_NAME} messages  ${PROTOBUF_LIBRARY} ${gRPC})

add_executable(${PROJECT_NAME}_reader "reader/main.cpp")
target_link_libraries(${PROJECT_NAME}_reader messages ${PROTOBUF_LIBRARY} grpc++)