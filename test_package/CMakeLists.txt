cmake_minimum_required(VERSION 3.1)
project(test_package)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup(TARGETS)

if(MSVC)
  add_definitions(-D_WIN32_WINNT=0x600)
endif()
set(CMAKE_CXX_STANDARD 11)

# Find Protobuf installation

find_package(Protobuf CONFIG REQUIRED)
find_package(gRPC CONFIG REQUIRED)

get_property(CPP_PLUGIN TARGET gRPC::grpc_cpp_plugin PROPERTY LOCATION)

message(STATUS "CPP_PLUGIN: ${CPP_PLUGIN}")

add_executable(greeter_client_server helloworld.proto greeter_client_server.cc)
protobuf_generate(
    APPEND_PATH
    LANGUAGE cpp
    TARGET greeter_client_server)
protobuf_generate(
    APPEND_PATH
    LANGUAGE grpc
    PLUGIN "protoc-gen-grpc=${CPP_PLUGIN}"
    TARGET greeter_client_server
    OUT_VAR GENERATED_SRC
    GENERATE_EXTENSIONS ".grpc.pb.h" ".grpc.pb.cc"
    )
target_link_libraries(greeter_client_server CONAN_PKG::protobuf CONAN_PKG::grpc)
target_include_directories(greeter_client_server PRIVATE ${CMAKE_CURRENT_BINARY_DIR})

message(STATUS "GENERATED_SRC: ${GENERATED_SRC}")
