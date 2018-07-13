cmake_minimum_required(VERSION 3.1)

project(vkcube)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)

set(VKCUBE_SPV
  "${CMAKE_CURRENT_SOURCE_DIR}/vkcube.frag.spv.h"
  "vkcube.vert.spv.h"
)

find_program(GLSLC glslc)
# http://fujii.github.io/2015/10/10/cmake-best-practice/
if(GLSLC)
    add_custom_command(
        OUTPUT ${VKCUBE_SPV}
        COMMAND ${GLSLC} vkcube.frag -mfmt=num -o vkcube.frag.spv.h)
    # https://gist.github.com/socantre/7ee63133a0a3a08f3990
else()
    add_custom_command(
        OUTPUT ${VKCUBE_SPV}
        COMMAND ${CMAKE_COMMAND} -E cp)
endif()
add_custom_target(vkcube_spv DEPENDS ${VKCUBE_SPV})

# Extra-cmake-modules
find_package(ECM REQUIRED NO_MODULE)
set(CMAKE_MODULE_PATH ${ECM_MODULE_PATH} ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules)

# Dependencies
find_package(Vulkan REQUIRED)
find_package(Wayland REQUIRED)
find_package(Libdrm REQUIRED)
find_package(PNG REQUIRED)
find_package(XCB REQUIRED)

# Sources & headers
file(GLOB SRC *.c)
file(GLOB INC *.h)
add_executable(${PROJECT_NAME} main.c)
add_dependencies(${PROJECT_NAME} vkcube_spv)
target_sources(${PROJECT_NAME} PRIVATE ${SRC})
target_include_directories(${PROJECT_NAME} PRIVATE ${INC}
    ${Vulkan_INCLUDE_DIRS} ${Wayland_INCLUDE_DIRS} ${Libdrm_INCLUDE_DIRS} ${PNG_INCLUDE_DIRS} ${XCB_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME}
    ${Vulkan_LIBRARIES} ${Wayland_LIBRARIES} ${Libdrm_LIBRARIES} ${PNG_LIBRARIES} ${XCB_LIBRARIES})

if (MSVC)
    add_compile_options(/M4)
else()
    add_compile_options(-Wall -Wextra -Wpedantic)
endif()
