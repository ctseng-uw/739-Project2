cmake_minimum_required(VERSION 3.15)
project(Fs)

set(CMAKE_CXX_STANDARD 20)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}" ${CMAKE_MODULE_PATH})
include(FetchContent)

fetchcontent_declare(
  gRPC
  GIT_REPOSITORY https://github.com/grpc/grpc
  GIT_TAG v1.44.0
)
set(FETCHCONTENT_QUIET OFF)
fetchcontent_makeavailable(gRPC)


add_custom_command(
  OUTPUT includes/hello.pb.cc includes/hello.grpc.pb.cc
  COMMAND ./gen_proto.sh
  DEPENDS protos/hello.proto protoc
)


add_executable(server server.cpp includes/hello.pb.cc includes/hello.grpc.pb.cc)
target_link_libraries(server grpc++)


find_package(FUSE REQUIRED)
include_directories(${FUSE_INCLUDE_DIR})
add_definitions("-D_FILE_OFFSET_BITS=64")
find_package(OpenSSL REQUIRED)
add_executable(client client.cpp client_grpc.cpp includes/hello.pb.cc includes/hello.grpc.pb.cc)
target_link_libraries(client grpc++)
target_link_libraries(client OpenSSL::SSL)
target_link_libraries(client ${FUSE_LIBRARIES})
