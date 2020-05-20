cmake_minimum_required(VERSION 3.10)
project(FLAMEGPU)

# Makes NVCC use c++11 by default, applies to all targets
if(NOT DEFINED CMAKE_CUDA_STANDARD)
    set(CMAKE_CUDA_STANDARD 11)
    set(CMAKE_CUDA_STANDARD_REQUIRED ON)
endif()

add_subdirectory(examples/HostLayerFunction)