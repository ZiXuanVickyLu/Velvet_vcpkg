cmake_minimum_required(VERSION 3.13 FATAL_ERROR)
set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake")

## Use C++17
set (CMAKE_CXX_STANDARD 17)
set (CMAKE_CUDA_STANDARD 17)

project(Velvet LANGUAGES CXX CUDA)
set(SRC_PATH ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME})
#add external
find_package(CUDA REQUIRED)
if(NOT CUDA_FOUND)
    message(FATAL_ERROR "CUDA not found")
else()
    include_directories(${CUDA_INCLUDE_DIRS})
    link_directories(${CUDA_TOOLKIT_ROOT_DIR}/lib64)
    link_libraries(${CUDA_LIBRARIES})
    set(CUDA_ARCHITECTURES 86 CACHE STRING "CUDA architectures" FORCE)
set(CMAKE_CUDA_ARCHITECTURES ${CUDA_ARCHITECTURES} CACHE STRING "CUDA architectures" FORCE)
endif()
find_package(assimp CONFIG REQUIRED)
find_package(fmt CONFIG REQUIRED)
find_package(glad CONFIG REQUIRED)
find_package(glfw3 CONFIG REQUIRED)
find_package(glm CONFIG REQUIRED)
find_package(imgui CONFIG REQUIRED)

file(GLOB_RECURSE SRC
        ${SRC_PATH}/*.cu
        ${SRC_PATH}/*.cpp
        ${SRC_PATH}/*.h
        ${SRC_PATH}/*.cuh})
list(REMOVE_ITEM ${SRC} ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}/main.cpp)
set(INC ${CMAKE_CURRENT_SOURCE_DIR}/Velvet)
include_directories(${INC})

add_library(${PROJECT_NAME}_lib STATIC ${SRC})
target_link_libraries(${PROJECT_NAME}_lib PUBLIC assimp::assimp fmt::fmt glad::glad glfw glm::glm imgui::imgui)

target_compile_options(${PROJECT_NAME}_lib PUBLIC $<$<COMPILE_LANGUAGE:CUDA>:
        --extended-lambda
        --default-stream per-thread
        >)
add_compile_definitions(${PROJECT_NAME}_lib _USE_MATH_DEFINES)

add_executable(${PROJECT_NAME}
        ${CMAKE_CURRENT_SOURCE_DIR}/${PROJECT_NAME}/main.cpp
)
target_link_libraries(${PROJECT_NAME} PRIVATE ${PROJECT_NAME}_lib)



