cmake_minimum_required(VERSION 3.10)

# 设置CUDA架构
set(CMAKE_CUDA_ARCHITECTURES "86")
set(CMAKE_CUDA_COMPILER "/usr/local/cuda/bin/nvcc")
# set(cusparse_DIR "/usr/local/cuda-12.0/targets/x86_64-linux/lib/libcusparse.so")

# 设置项目名称和语言
project(dawn LANGUAGES CXX CUDA)

# 设置C++标准
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)


project(dawn_cpu_apsp LANGUAGES CXX)
project(dawn_cpu_sssp LANGUAGES CXX)
project(dawn_cpu_mssp LANGUAGES CXX)
project(dawn_gpu_apsp LANGUAGES CXX CUDA)
project(dawn_gpu_sssp LANGUAGES CXX CUDA)
project(dawn_gpu_mssp LANGUAGES CXX CUDA)
# project(test LANGUAGES CXX)
# project(test_cpu LANGUAGES CXX CUDA)

# 添加头文件路径
include_directories(include)
find_package(CUDA REQUIRED)
include_directories(${CUDA_INCLUDE_DIRS})

# 添加可执行目标
add_executable(dawn_cpu_apsp apsp/dawn_cpu_apsp.cpp include/dawn.hxx include/tool.cpp include/graph.cpp include/run.cpp include/access.h)
add_executable(dawn_cpu_sssp sssp/dawn_cpu_sssp.cpp include/dawn.hxx include/tool.cpp include/graph.cpp include/run.cpp include/access.h)
add_executable(dawn_cpu_mssp mssp/dawn_cpu_mssp.cpp include/dawn.hxx include/tool.cpp include/graph.cpp include/run.cpp include/access.h)
add_executable(dawn_gpu_apsp apsp/dawn_gpu_apsp.cu include/dawn.cuh include/dawn.hxx include/tool.cpp include/graph.cpp include/access.h)
add_executable(dawn_gpu_sssp sssp/dawn_gpu_sssp.cu include/dawn.cuh include/dawn.hxx include/tool.cpp include/graph.cpp include/access.h)
add_executable(dawn_gpu_mssp mssp/dawn_gpu_mssp.cu include/dawn.cuh include/dawn.hxx include/tool.cpp include/graph.cpp include/access.h)

# add_executable(test test/test.cu include/dawn.cuh include/dawn.hxx include/tool.cpp include/graph.cpp include/access.h)
# add_executable(test_cpu test/test.cpp include/dawn.hxx include/tool.cpp include/graph.cpp include/run.cpp include/access.h)

# 设置编译选项
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -O3 -gencode arch=compute_86,code=sm_86")
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -Xcompiler -fopenmp")

# 添加OpenMP支持
find_package(OpenMP REQUIRED)
if(OPENMP_FOUND)
    target_link_libraries(dawn_gpu_apsp PUBLIC OpenMP::OpenMP_CXX)
    target_link_libraries(dawn_cpu_apsp PUBLIC OpenMP::OpenMP_CXX)
    target_link_libraries(dawn_cpu_sssp PUBLIC OpenMP::OpenMP_CXX)
    target_link_libraries(dawn_gpu_sssp PUBLIC OpenMP::OpenMP_CXX)
    target_link_libraries(dawn_cpu_mssp PUBLIC OpenMP::OpenMP_CXX)
    target_link_libraries(dawn_gpu_mssp PUBLIC OpenMP::OpenMP_CXX)
# target_link_libraries(test PUBLIC OpenMP::OpenMP_CXX)
# target_link_libraries(test_cpu PUBLIC OpenMP::OpenMP_CXX)
endif()

# target_link_libraries(dawn_cpu_v1 PUBLIC ${cusparse_DIR})
# target_link_libraries(dawn_gpu_v1 PUBLIC ${cusparse_DIR})
# target_link_libraries(dawn_gpu_sssp PUBLIC ${cusparse_DIR})
# target_link_libraries(spmv PUBLIC ${cusparse_DIR})
# target_link_libraries(test PUBLIC ${cusparse_DIR})







