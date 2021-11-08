cmake_minimum_required(VERSION 3.0)

project(yolov5)

add_definitions(-std=c++11)
add_compile_definitions(
    API_EXPORTS
    NOMINMAX
)
option(CUDA_USE_STATIC_CUDA_RUNTIME OFF)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE Debug)

find_package(CUDA REQUIRED)

if(WIN32)
    #enable_language(CUDA)
    set(TENSORRT_GPU_ROOT "C:/TensorRT-8.2.0.6")
    set(OPENCV_ROOT "c:/opencv/build")
endif(WIN32)

find_package(CUDAToolkit)
if (CUDAToolkit_FOUND)
    message(STATUS "Found CUDAToolkit")
    message(STATUS "CUDAToolkit_VERSION     : ${CUDAToolkit_VERSION}")
    message(STATUS "CUDAToolkit_INCLUDE_DIRS: ${CUDAToolkit_INCLUDE_DIRS}")
else()
    message(FATAL_ERROR "Could not find CUDAToolkit")
endif()


include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(.)
# include and link dirs of cuda and tensorrt, you need adapt them if yours are different
# cuda
include_directories(/usr/local/cuda/include)
link_directories(/usr/local/cuda/lib64)
# tensorrt
include_directories(/usr/include/x86_64-linux-gnu/)
link_directories(/usr/lib/x86_64-linux-gnu/)


if (UNIX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Ofast -Wfatal-errors -D_MWAITXINTRIN_H_INCLUDED")
endif()

cuda_add_library(myplugins SHARED ${PROJECT_SOURCE_DIR}/yololayer.cu)

target_link_libraries(myplugins 
    nvinfer 
    CUDA::cudart
)

if (WIN32)
    set(CMAKE_PREFIX_PATH ${OPENCV_ROOT})
endif()

find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

add_executable(yolov5 ${PROJECT_SOURCE_DIR}/calibrator.cpp ${PROJECT_SOURCE_DIR}/yolov5.cpp)

target_link_libraries(yolov5 nvinfer)
target_link_libraries(yolov5 CUDA::cudart)
target_link_libraries(yolov5 myplugins)
target_link_libraries(yolov5 ${OpenCV_LIBS})

if (WIN32)
    target_include_directories(myplugins
        PRIVATE ${TENSORRT_GPU_ROOT}/include
    )
    target_include_directories(yolov5
        PRIVATE ${TENSORRT_GPU_ROOT}/include
    )
    target_link_directories(myplugins 
        PRIVATE ${TENSORRT_GPU_ROOT}/lib
    )
    target_link_directories(yolov5 
        PRIVATE ${TENSORRT_GPU_ROOT}/lib
    )
endif()


if(UNIX)
add_definitions(-O2 -pthread)
endif(UNIX)

