cmake_minimum_required(VERSION 3.5)
project(serial VERSION 1.2.1 LANGUAGES CXX)

# Include GNUInstallDirs for standard installation directories
include(GNUInstallDirs)

# Set C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Options
option(BUILD_EXAMPLES "Build serial port examples" ON)
option(BUILD_SHARED_LIBS "Build shared libraries instead of static" ON)

# Find required libraries for Linux
set(rt_LIBRARIES rt)
set(pthread_LIBRARIES pthread)

## Sources
set(serial_SRCS
    src/serial.cc
    src/impl/unix.cc
    src/impl/list_ports/list_ports_linux.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)

## Add serial library
add_library(${PROJECT_NAME} ${serial_SRCS})

# Set library properties
set_target_properties(${PROJECT_NAME} PROPERTIES
    VERSION ${PROJECT_VERSION}
    SOVERSION ${PROJECT_VERSION_MAJOR}
    PUBLIC_HEADER "include/serial/serial.h;include/serial/v8stdint.h"
    POSITION_INDEPENDENT_CODE ON
)

# Include directories
target_include_directories(${PROJECT_NAME}
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

# Link libraries
target_link_libraries(${PROJECT_NAME} rt pthread)

## Build example
if(BUILD_EXAMPLES)
    add_executable(serial_example examples/serial_example.cc)
    target_link_libraries(serial_example ${PROJECT_NAME})
endif()

## Install targets
install(TARGETS ${PROJECT_NAME}
    EXPORT serialTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/serial
)

## Install headers
install(DIRECTORY include/serial
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    FILES_MATCHING PATTERN "*.h"
)

## Export targets
install(EXPORT serialTargets
    FILE serialTargets.cmake
    NAMESPACE serial::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/serial
)

## Create config file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/serialConfigVersion.cmake"
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/serialConfig.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/serialConfig.cmake"
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/serial
    PATH_VARS CMAKE_INSTALL_INCLUDEDIR CMAKE_INSTALL_LIBDIR
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/serialConfig.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/serialConfigVersion.cmake"
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/serial
)

# Create uninstall target
if(NOT TARGET uninstall)
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        IMMEDIATE @ONLY)
    
    add_custom_target(uninstall
        COMMAND ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
endif()

# Print installation summary
message(STATUS "")
message(STATUS "Serial library configuration (Linux):")
message(STATUS "  Version: ${PROJECT_VERSION}")
message(STATUS "  Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "  Library type: ${BUILD_SHARED_LIBS}")
message(STATUS "  Position independent code: ON")
message(STATUS "  Install prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "  Library destination: ${CMAKE_INSTALL_FULL_LIBDIR}")
message(STATUS "  Include destination: ${CMAKE_INSTALL_FULL_INCLUDEDIR}")
message(STATUS "  Build examples: ${BUILD_EXAMPLES}")
message(STATUS "")
