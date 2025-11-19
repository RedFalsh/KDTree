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

# ================================
# 安装规则
# ================================
include(GNUInstallDirs)

# 1. 安装头文件
install(FILES ${KDTREE_HEADERS}
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    COMPONENT headers
)

# 2. 安装库文件（.lib/.a/.so/.dylib）
install(TARGETS KDTree
    EXPORT KDTreeTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}     # .so/.dylib
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}      # .lib/.a
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}      # .dll (Windows only)
    COMPONENT libraries
)

# ================================
# 导出 CMake 配置包（支持 find_package）
# ================================
include(CMakePackageConfigHelpers)

# 生成版本文件
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/KDTreeConfigVersion.cmake"
    VERSION 2.0.1
    COMPATIBILITY AnyNewerVersion
)

# 生成配置文件（手动或用 configure_package_config_file）
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/KDTreeConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/KDTreeConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/KDTree
)

# 安装 Targets 和 Config
install(EXPORT KDTreeTargets
    FILE KDTreeTargets.cmake
    NAMESPACE KDTree::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/KDTree
    COMPONENT devel
)

install(
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/KDTreeConfig.cmake"
        "${CMAKE_CURRENT_BINARY_DIR}/KDTreeConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/KDTree
    COMPONENT devel
)
