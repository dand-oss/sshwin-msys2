cmake_minimum_required(VERSION 3.30)
project(sshwin VERSION 1.0.0 LANGUAGES CXX)

set(exename "sshwin-msys2")

###############
# ASI default settings
list(APPEND CMAKE_MODULE_PATH "$ENV{ASV_CMAKE}")
include(asv_cmake_defaults)
asv_cmake_defaults( )
###############

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# C++17 required for std::regex, std::thread
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(${exename} WIN32)
target_sources(${exename} PRIVATE
    sshwin-msys2/main.cpp
    sshwin-msys2/socketServer.cpp
    sshwin-msys2/socketServer.h
    sshwin-msys2/pch.h
    sshwin-msys2/LimitSingleInstance.h
    sshwin-msys2/resource.h
)

target_include_directories(${exename} PRIVATE sshwin-msys2)

# Windows-specific configuration
if(WIN32)
    target_compile_definitions(${exename} PRIVATE
        WIN32_LEAN_AND_MEAN
        _CRT_SECURE_NO_WARNINGS
    )
    target_link_libraries(${exename} PRIVATE
        Ws2_32
        ole32  # For CoCreateGuid
    )
endif()

# Installation
install(TARGETS ${exename}
    EXPORT sshwinTargets
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# Export targets
install(EXPORT sshwinTargets
    FILE sshwinTargets.cmake
    NAMESPACE sshwin::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/sshwin
)

# Generate version file
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/sshwinConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

# Configure and install sshwinConfig.cmake
configure_package_config_file(
    "${CMAKE_SOURCE_DIR}/sshwinConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/sshwinConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/sshwin
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/sshwinConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/sshwinConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/sshwin
)

# CPack configuration
set(CPACK_PACKAGE_NAME "sshwin")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Windows SSH-agent proxy for MSYS2/Git-Bash/WSL2")
set(CPACK_PACKAGE_VENDOR "AppSmiths Ventures LLP")
set(CPACK_GENERATOR "ZIP;TGZ")
include(CPack)
