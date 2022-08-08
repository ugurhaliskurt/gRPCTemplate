cmake_minimum_required(VERSION 3.13)

project(gRPCexample LANGUAGES CXX)

set(CMAKE_BUILD_TYPE Debug)

include(FindProtobuf)
find_package(Protobuf REQUIRED)
include_directories(${CMAKE_BINARY_DIR})

file(GLOB ProtoFiles "${CMAKE_CURRENT_SOURCE_DIR}/src/proto/*.proto")
PROTOBUF_GENERATE_CPP(ProtoSources ProtoHeaders ${ProtoFiles})
add_library(messages STATIC ${ProtoSources} ${ProtoHeaders})

add_executable(${PROJECT_NAME} "src/main.cpp")
target_link_libraries(${PROJECT_NAME} messages ${PROTOBUF_LIBRARY})

add_executable(${PROJECT_NAME}_reader "reader/main.cpp")
target_link_libraries(${PROJECT_NAME}_reader messages ${PROTOBUF_LIBRARY} )