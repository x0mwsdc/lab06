cmake_minimum_required(VERSION 3.10)
project(print_project)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

include_directories(include)

add_library(print sources/print.cpp)

add_executable(example1 examples/example1.cpp)
target_link_libraries(example1 print)

add_executable(example2 examples/example2.cpp)
target_link_libraries(example2 print)
