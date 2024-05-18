cmake_minimum_required(VERSION 3.20)
project(zdepth LANGUAGES CXX)
include("cmake/zdepth-config-version.cmake")
set(CMAKE_CXX_STANDARD 11)

include(GNUInstallDirs)

################################################################################
# Build Options

set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "" FORCE)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# In debug mode, add -DDEBUG
add_compile_options("$<$<CONFIG:DEBUG>:-DDEBUG>")

if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
else()
    # Warnings
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -pedantic")

    # Remove Asio warning
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-local-typedefs")

    # Static library: -fPIC
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

    set(CMAKE_CXX_FLAGS_DEBUG "-g -O0 -march=native -fstack-protector")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3 -march=native")
endif()


################################################################################
# Subprojects

find_package(zstd REQUIRED)


set(ZDEPTH_BUILD_STATIC ON)
if (BUILD_SHARED_LIBS)
    set(ZDEPTH_BUILD_STATIC OFF)
endif()
configure_file(${PROJECT_SOURCE_DIR}/include/zdepth_config.h.in ${CMAKE_BINARY_DIR}/include/zdepth_config.h @ONLY)

################################################################################
# Source

set(INCLUDE_FILES
    include/zdepth.hpp
    ${CMAKE_BINARY_DIR}/include/zdepth_config.h
)

set(SOURCE_FILES
    ${INCLUDE_FILES}
    src/zdepth.cpp
)


################################################################################
# Targets

# zdepth library


add_library(zdepth ${SOURCE_FILES} ${INCLUDE_FILES})
add_library(zdepth::zdepth ALIAS zdepth)

if (BUILD_SHARED_LIBS)
  set_target_properties(zdepth PROPERTIES COMPILE_DEFINITIONS "ZDEPTH_EXPORTS")
endif()
target_link_libraries(zdepth PUBLIC zstd::zstd)

target_include_directories(zdepth PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:include>"
    PRIVATE
    ${PROJECT_BINARY_DIR}/src
    ${CMAKE_BINARY_DIR}/include
)


################################################################################
# Install

install(TARGETS zdepth EXPORT zdepthConfig
        ARCHIVE  DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY  DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME  DESTINATION ${CMAKE_INSTALL_BINDIR})  # This is for Windows

install(FILES LICENSE DESTINATION ${CMAKE_INSTALL_PREFIX})
install(FILES README.md DESTINATION ${CMAKE_INSTALL_PREFIX})
install(DIRECTORY cmake DESTINATION ${CMAKE_INSTALL_PREFIX})
install(FILES include/zdepth.hpp DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(FILES ${CMAKE_BINARY_DIR}/include/zdepth_config.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})


# zdepth test application

add_executable(zdepth_test tests/zdepth_test.cpp)
target_link_libraries(zdepth_test PRIVATE zdepth)

target_include_directories(zdepth_test
        PRIVATE
        ${PROJECT_SOURCE_DIR}/include
        ${PROJECT_BINARY_DIR}/src
        ${PROJECT_SOURCE_DIR}/src
        ${CMAKE_BINARY_DIR}/include
        )

if (BUILD_SHARED_LIBS)
  target_link_libraries(zdepth_test PUBLIC zstd::libzstd_shared)
else()
  set_target_properties(zdepth_test PROPERTIES COMPILE_DEFINITIONS "ZDEPTH_BUILD_STATIC")
  target_link_libraries(zdepth_test PUBLIC zstd::libzstd_static)
endif()


