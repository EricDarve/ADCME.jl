cmake_minimum_required(VERSION 3.5)
project(TF_CUSTOM_OP)
set (CMAKE_CXX_STANDARD 11)

execute_process(COMMAND julia -e "import ADCME; print(ADCME.__STR__)" OUTPUT_VARIABLE JL_OUT)
execute_process(COMMAND which julia OUTPUT_VARIABLE JULIA_BIN)
message("Julia=${JULIA_BIN}")

list(GET JL_OUT 0 BINDIR)
list(GET JL_OUT 1 LIBDIR)
list(GET JL_OUT 2 TF_INC)
list(GET JL_OUT 3 TF_ABI)
list(GET JL_OUT 4 EIGEN_INC)
list(GET JL_OUT 5 CC)
list(GET JL_OUT 6 CXX)
list(GET JL_OUT 7 CMAKE)
list(GET JL_OUT 8 MAKE)
list(GET JL_OUT 9 GIT)
list(GET JL_OUT 10 PYTHON)
list(GET JL_OUT 11 TF_LIB_FILE)

message("Python path=${PYTHON}")
message("EIGEN_INC=${EIGEN_INC}")
message("TF_INC=${TF_INC}")
message("TF_ABI=${TF_ABI}")
message("TF_LIB_FILE=${TF_LIB_FILE}")


if (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 5.0 OR CMAKE_CXX_COMPILER_VERSION VERSION_EQUAL 5.0)
  set(CMAKE_CXX_FLAGS "-D_GLIBCXX_USE_CXX11_ABI=${TF_ABI} ${CMAKE_CXX_FLAGS}")
endif()

set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")

include_directories(${TF_INC} ${EIGEN_INC}/../../include ${EIGEN_INC})
link_directories(${TF_LIB})

#find_package(CUDA QUIET)
#set(CMAKE_CXX_FLAGS "-std=c++11 ${CMAKE_CXX_FLAGS}")
#set(CMAKE_CXX_FLAGS "-O3 ${CMAKE_CXX_FLAGS}")
#set(CMAKE_CXX_FLAGS "-shared ${CMAKE_CXX_FLAGS}")
#set(CMAKE_CXX_FLAGS "-fPIC ${CMAKE_CXX_FLAGS}")
#set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};--expt-relaxed-constexpr)
#SET(CUDA_PROPAGATE_HOST_FLAGS ON)

#find_program(_nvidia_smi "nvidia-smi")
#if (_nvidia_smi)
#  add_definitions(-DUSE_GPU)
#  message("Compiling GPU-compatible custom operator!")
#  cuda_add_library(SparseFactorization SHARED SparseFactorization.cpp SparseFactorization.cu)
#else()
  
#endif()

add_library(lru SHARED lru_cache.cpp)
set_property(TARGET lru PROPERTY POSITION_INDEPENDENT_CODE ON)
target_link_libraries(lru ${TF_LIB_FILE})


add_library(factorization SHARED Factorization/SparseFactorization.cpp)
set_property(TARGET factorization PROPERTY POSITION_INDEPENDENT_CODE ON)
target_link_libraries(factorization ${TF_LIB_FILE} lru)


add_library(Solve SHARED Solve/Solve.cpp)
set_property(TARGET Solve PROPERTY POSITION_INDEPENDENT_CODE ON)
target_link_libraries(Solve ${TF_LIB_FILE} lru)