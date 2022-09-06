
cmake_minimum_required(VERSION 3.18 FATAL_ERROR)


project(MNIST
        LANGUAGES CXX CUDA)

find_package(CUDAToolkit)
find_package(CUDA)
find_package(OpenCV)


# TensorRT
# Include Directories
find_path(TENSORRT_INCLUDE_DIRS NvInfer.h
    HINTS ${TENSORRT_ROOT} ${CUDA_TOOLKIT_ROOT_DIR}
    PATH_SUFFIXES include)
MESSAGE(STATUS "Found TensorRT Nvinfer.h at ${TENSORRT_INCLUDE_DIRS}")
find_path(TENSORRT_TARGETS_INCLUDE_DIR cuda_runtime_api.h
    HINTS ${TENSORRT_ROOT} ${CUDA_TOOLKIT_ROOT_DIR}
    PATH_SUFFIXES targets/x86_64-linux/include targets/aarch64-linux/include)
MESSAGE(STATUS "Found TensorRT cuda_runtime_api.h at ${TENSORRT_TARGETS_INCLUDE_DIR}")
list(APPEND TENSORRT_INCLUDE_DIRS ${TENSORRT_TARGETS_INCLUDE_DIR})
# Libraries
find_library(TENSORRT_LIBRARY_INFER nvinfer
    HINTS ${TENSORRT_ROOT} ${TENSORRT_BUILD} ${CUDA_TOOLKIT_ROOT_DIR}
    PATH_SUFFIXES lib lib64 lib/x64)
find_library(TENSORRT_LIBRARY_INFER_PLUGIN nvinfer_plugin
    HINTS  ${TENSORRT_ROOT} ${TENSORRT_BUILD} ${CUDA_TOOLKIT_ROOT_DIR}
    PATH_SUFFIXES lib lib64 lib/x64)
set(TENSORRT_LIBRARIES ${TENSORRT_LIBRARY_INFER} ${TENSORRT_LIBRARY_INFER_PLUGIN})
MESSAGE(STATUS "Found TensorRT libs at ${TENSORRT_LIBRARIES}")
find_package_handle_standard_args(
    TENSORRT DEFAULT_MSG TENSORRT_INCLUDE_DIRS TENSORRT_LIBRARIES)
if(NOT TENSORRT_FOUND)
    message(ERROR "Cannot find TensorRT library.")
endif()

# ONNX Libraries
find_library(ONNX_LIBRARIES nvonnxparser
    HINTS ${TENSORRT_ROOT} ${TENSORRT_BUILD} ${CUDA_TOOLKIT_ROOT_DIR}
    PATH_SUFFIXES lib lib64 lib/x64)
MESSAGE(STATUS "Found nvonnxparser at ${ONNX_LIBRARIES}")

# Local
# Include Directory
set(LOCAL_INCLUDE_DIRS "${CMAKE_SOURCE_DIR}/common")
# Libraries
add_library(logger "${LOCAL_INCLUDE_DIRS}/logger.cpp")

set(LOCAL_LIBS logger)


include_directories($(CUDA_INCLUDE_DIRS))
message(STATUS "----------------------------------------")
message(STATUS "Include Directories: ")
message(STATUS "CUDA_INCLUDE_DIRS = ${CUDA_INCLUDE_DIRS}")
message(STATUS "ONNX_INCLUDE_DIRS = ${ONNX_INCLUDE_DIRS}")
message(STATUS "TENSORRT_INCLUDE_DIRS = ${TENSORRT_INCLUDE_DIRS}")
message(STATUS "OpenCV_INCLUDE_DIRS = ${OpenCV_INCLUDE_DIRS}")
message(STATUS "LOCAL_INCLUDE_DIRS = ${LOCAL_INCLUDE_DIRS}")
message(STATUS "----------------------------------------")
message(STATUS "Libraries for Linking: ")
message(STATUS "ONNX_LIBRARIES = ${ONNX_LIBRARIES}")
message(STATUS "CUDA_LIBRARIES = ${CUDA_LIBRARIES}")
message(STATUS "TENSORRT_LIBRARIES = ${TENSORRT_LIBRARIES}")
message(STATUS "OpenCV_LIBS = ${OpenCV_LIBS}")
message(STATUS "----------------------------------------")


add_executable(inference mnist_inference.cpp)



target_include_directories(logger PUBLIC 
    ${TENSORRT_INCLUDE_DIRS} 
    ${CUDA_INCLUDE_DIRS} 
)

target_link_libraries(logger PUBLIC
    ${TENSORRT_LIBRARIES}
    ${CUDA_LIBRARIES} 
)

target_include_directories(inference PUBLIC 
    ${ONNX_INCLUDE_DIRS} 
    ${TENSORRT_INCLUDE_DIRS} 
    ${CUDA_INCLUDE_DIRS} 
    ${OpenCV_INCLUDE_DIRS}
    ${LOCAL_INCLUDE_DIRS}
)

target_link_libraries(inference PUBLIC
    ${ONNX_LIBRARIES}
    ${TENSORRT_LIBRARIES}
    ${CUDA_LIBRARIES} 
    ${OpenCV_LIBS}
    ${LOCAL_LIBS}
)


