cmake_minimum_required(VERSION 3.15)
project(MyProject)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_FLAGS_DEBUG "-O0 -g -fsanitize=address")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")

include(FetchContent)

fetchcontent_declare(
  gRPC
  GIT_REPOSITORY https://github.com/grpc/grpc
  GIT_TAG v1.49.2
)
set(FETCHCONTENT_QUIET OFF)
fetchcontent_makeavailable(gRPC) 

file(GLOB PROTOBUF_SRC
     "protos/*.proto"
)

set(PROTOBUF_DST "")
set(PROTOBUF_EXT ".grpc.pb.cc" ".grpc.pb.h" ".pb.cc" ".pb.h")
foreach(file ${PROTOBUF_SRC})
  foreach(ext ${PROTOBUF_EXT})
    get_filename_component(fileh ${file} NAME_WE)
    string(PREPEND fileh "${CMAKE_CURRENT_SOURCE_DIR}/.proto_gen/")
    string(APPEND fileh ${ext})
    list(APPEND PROTOBUF_DST ${fileh})
  endforeach()
endforeach()

add_custom_command(
  OUTPUT ${PROTOBUF_DST}
  COMMAND ${grpc_BINARY_DIR}/third_party/protobuf/protoc
  --proto_path=${CMAKE_CURRENT_SOURCE_DIR}/protos
  --cpp_out=${CMAKE_CURRENT_SOURCE_DIR}/.proto_gen
  --grpc_out=${CMAKE_CURRENT_SOURCE_DIR}/.proto_gen
  --plugin=protoc-gen-grpc=${grpc_BINARY_DIR}/grpc_cpp_plugin
  ${PROTOBUF_SRC}
  DEPENDS ${PROTOBUF_SRC} protoc grpc_cpp_plugin grpc++_reflection
)

add_executable(client client.cpp ${PROTOBUF_DST})
target_include_directories(client PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/.proto_gen)
target_link_libraries(client grpc++)

add_executable(server server.cpp ${PROTOBUF_DST})
target_include_directories(server PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/.proto_gen)
target_link_libraries(server grpc++ grpc++_reflection)
