# Minimum CMake required
cmake_minimum_required(VERSION 3.10)

# Project
project(grpc-examples CXX)

# Pthreads
find_package(Threads REQUIRED)

# Protobuf
set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(Protobuf REQUIRED)
message(STATUS "Using protobuf ${protobuf_VERSION}")

# CPP compile flags
#set c++ standard   
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(SUBDIRS sync async)

foreach(dir ${SUBDIRS})
    add_subdirectory(${dir})
endforeach()

