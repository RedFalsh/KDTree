cmake_minimum_required(VERSION 3.14)
project(KDTree LANGUAGES CXX)

# ================================
# 编译器标志：区分 MSVC 和 GNU/Clang
# ================================
if(MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE
        /W4 /WX
        /std:c++17
        # 可选：/wd4251 /wd4275 抑制 dll 导出模板警告（若用 STL 类导出）
    )
else()
    target_compile_options(${PROJECT_NAME} PRIVATE
        -Wall -Wextra -Werror
        -std=c++17
    )
    set(CMAKE_CXX_FLAGS_DEBUG          "-g")
    set(CMAKE_CXX_FLAGS_MINSIZEREL     "-Os -DNDEBUG")
    set(CMAKE_CXX_FLAGS_RELEASE        "-O3 -DNDEBUG")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g")
endif()

if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type" FORCE)
    message(STATUS "Build type not specified; defaulting to 'Release'")
endif()
message(STATUS "Using build type: ${CMAKE_BUILD_TYPE}")

# ================================
# 输出目录
# ================================
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# ================================
# 主库定义
# ================================
add_library(KDTree SHARED KDTree.cpp)

# 假设头文件为 KDTree.hpp（请按实际调整！）
set(KDTREE_HEADERS KDTree.hpp)  # ← 核心头文件，若多个可 list(APPEND ...)

target_include_directories(KDTree PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)

# 可选：为 MSVC 添加 DLL 导出宏支持（推荐）
if(MSVC OR MINGW)
    target_compile_definitions(KDTree PRIVATE KDTree_EXPORTS)
endif()
