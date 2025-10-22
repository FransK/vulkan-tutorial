cmake_minimum_required(VERSION 3.28)

# set the project name
project(Tutorial)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Find required packages
find_package(PkgConfig REQUIRED)
find_package(Vulkan REQUIRED)

# Find GLFW
pkg_check_modules(GLFW3 REQUIRED glfw3)

# add the source directory
include_directories(src)

# add the executable
add_executable(Tutorial src/main.cpp)

# Set compiler flags for debug and release builds
target_compile_options(Tutorial PRIVATE
    $<$<CONFIG:DEBUG>:-g -O0 -Wall -Wextra -Wpedantic>
    $<$<CONFIG:RELEASE>:-O3 -Wall -Wextra -Wpedantic>
)

# Link libraries
target_link_libraries(Tutorial ${Vulkan_LIBRARIES} ${GLFW3_LIBRARIES})
target_include_directories(Tutorial PRIVATE ${Vulkan_INCLUDE_DIRS} ${GLFW3_INCLUDE_DIRS})
target_compile_definitions(Tutorial PRIVATE ${GLFW3_CFLAGS_OTHER} VULKAN_HPP_NO_STRUCT_CONSTRUCTORS)