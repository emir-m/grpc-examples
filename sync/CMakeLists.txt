set(PROTO_PATH "${CMAKE_CURRENT_SOURCE_DIR}/protos")
set(GENERATED_PROTOBUF_PATH "${CMAKE_CURRENT_BINARY_DIR}/generated")
file(MAKE_DIRECTORY ${GENERATED_PROTOBUF_PATH})

# Generated grpc and proto sources
set(COMPANY_PROTO "${PROTO_PATH}/company.proto")
set(COMPANY_PROTO_HEADER "${GENERATED_PROTOBUF_PATH}/company.pb.h")
set(COMPANY_PROTO_SRC "${GENERATED_PROTOBUF_PATH}/company.pb.cc")
set(COMPANY_GRPC_HEADER "${GENERATED_PROTOBUF_PATH}/company.grpc.pb.h")
set(COMPANY_GRPC_SRC "${GENERATED_PROTOBUF_PATH}/company.grpc.pb.cc")

find_program(PROTOC NAMES protoc protoc.exe)
MESSAGE(STATUS "PROTOC: ${PROTOC}")

add_custom_command(
    OUTPUT "${COMPANY_PROTO_SRC}"
           "${COMPANY_PROTO_HEADER}"
    COMMAND ${PROTOC}
    ARGS -I "${PROTO_PATH}"
         --cpp_out="${GENERATED_PROTOBUF_PATH}"
         "${COMPANY_PROTO}"
    DEPENDS "${COMPANY_PROTO}"
)

find_program(PROTOC_GEN_GRPC_CPP NAMES protoc-gen-grpc-cpp grpc_cpp_plugin.exe)
MESSAGE(STATUS "PROTOC_GEN_GRPC_CPP: ${PROTOC_GEN_GRPC_CPP}")


add_custom_command(
    OUTPUT "${COMPANY_GRPC_SRC}"
           "${COMPANY_GRPC_HEADER}"
    COMMAND ${PROTOC}
    ARGS --grpc_out "${GENERATED_PROTOBUF_PATH}" -I "${PROTO_PATH}"
         --plugin=protoc-gen-grpc="${PROTOC_GEN_GRPC_CPP}"
         "${COMPANY_PROTO}"
    DEPENDS "${COMPANY_PROTO}"
)

# Generated include directory
include_directories("${GENERATED_PROTOBUF_PATH}")

find_package(gRPC CONFIG REQUIRED)
find_package(protobuf CONFIG REQUIRED)

file(GLOB PROTO_FILES "${PROTO_PATH}/*.proto")
SOURCE_GROUP("Proto files" FILES ${PROTO_FILES})

foreach(_target
  company_server company_client)
  add_executable(${_target} "${_target}.cc"
    ${COMPANY_PROTO_SRC}
    ${COMPANY_GRPC_SRC}
    ${PROTO_FILES}
  )
  target_link_libraries(${_target}
    protobuf::libprotoc protobuf::libprotobuf protobuf::libprotobuf-lite
    gRPC::grpc gRPC::grpc++ gRPC::grpc++_alts
)
endforeach()
