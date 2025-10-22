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

# Find Slang compiler (optional)
find_program(SLANGC_EXECUTABLE slangc)

if(SLANGC_EXECUTABLE)
    message(STATUS "Found Slang compiler: ${SLANGC_EXECUTABLE}")
else()
    message(WARNING "Slang compiler not found. Shader compilation will be skipped.")
endif()

# slang compiler function
function (add_slang_shader_target TARGET)
  cmake_parse_arguments ("SHADER" "" "" "SOURCES" ${ARGN})
  set (SHADERS_DIR ${CMAKE_CURRENT_LIST_DIR}/shaders)
  set (ENTRY_POINTS -entry vertMain -entry fragMain)
  add_custom_command (
          OUTPUT ${SHADERS_DIR}
          COMMAND ${CMAKE_COMMAND} -E make_directory ${SHADERS_DIR}
  )
  add_custom_command (
          OUTPUT  ${SHADERS_DIR}/slang.spv
          COMMAND ${SLANGC_EXECUTABLE} ${SHADER_SOURCES} -target spirv -profile spirv_1_4 -emit-spirv-directly -fvk-use-entrypoint-name ${ENTRY_POINTS} -o slang.spv
          WORKING_DIRECTORY ${SHADERS_DIR}
          DEPENDS ${SHADERS_DIR} ${SHADER_SOURCES}
          COMMENT "Compiling Slang Shaders"
          VERBATIM
  )
  add_custom_target (${TARGET} DEPENDS ${SHADERS_DIR}/slang.spv)
endfunction()

# add the source directory
include_directories(src)

# add the executable
add_executable(Tutorial src/main.cpp)

# Define shader sources
set(SHADER_SLANG_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/shaders/shader.slang)

# Add shader compilation target if slang compiler is available
if(SLANGC_EXECUTABLE)
    add_slang_shader_target(compile_shaders SOURCES ${SHADER_SLANG_SOURCES})
    add_dependencies(Tutorial compile_shaders)
endif()

# Set compiler flags for debug and release builds
target_compile_options(Tutorial PRIVATE
    $<$<CONFIG:DEBUG>:-g -O0 -Wall -Wextra -Wpedantic>
    $<$<CONFIG:RELEASE>:-O3 -Wall -Wextra -Wpedantic>
)

# Link libraries
target_link_libraries(Tutorial ${Vulkan_LIBRARIES} ${GLFW3_LIBRARIES})
target_include_directories(Tutorial PRIVATE ${Vulkan_INCLUDE_DIRS} ${GLFW3_INCLUDE_DIRS})
target_compile_definitions(Tutorial PRIVATE ${GLFW3_CFLAGS_OTHER} VULKAN_HPP_NO_STRUCT_CONSTRUCTORS)