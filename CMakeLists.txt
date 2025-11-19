cmake_minimum_required(VERSION 3.15)
project(KDTree LANGUAGES CXX)

# 由 Conan 或命令行传入编译标志，CMake 不硬编码
add_library(KDTree SHARED KDTree.cpp)

# 假设头文件是 KDTree.hpp（请按实际改）
target_include_directories(KDTree PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include>
)

# ———————— 安装规则（安全跨平台） ————————
include(GNUInstallDirs)

install(
    FILES KDTree.hpp
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
    TARGETS KDTree
    EXPORT KDTreeTargets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}  # .dll 放 bin/
)

# ———————— 支持 find_package ————————
include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/KDTreeConfigVersion.cmake"
    VERSION 2.0.1
    COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/KDTreeConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/KDTreeConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/KDTree
)

install(EXPORT KDTreeTargets
    FILE KDTreeTargets.cmake
    NAMESPACE KDTree::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/KDTree
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/KDTreeConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/KDTreeConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/KDTree
)
