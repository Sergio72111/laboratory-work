cmake_minimum_required(VERSION 3.10)

# Set project version
set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VERSION})

# Find Boost library
find_package(Boost COMPONENTS unit_test_framework REQUIRED)

# Configure version header file
configure_file(version.h.in version.h)

# Define executables and libraries
add_executable(helloworld_cli main.cpp)
add_library(helloworld lib.cpp)
add_executable(test_version test_version.cpp)

# Set C++ standard properties
set_target_properties(helloworld_cli helloworld test_version PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

# Include directories for helloworld library
target_include_directories(helloworld PRIVATE "${CMAKE_BINARY_DIR}")

# Set compile definitions and include directories for test_version
set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIRS} # Use Boost_INCLUDE_DIRS instead of Boost_INCLUDE_DIR
)

# Link libraries to executables
target_link_libraries(helloworld_cli PRIVATE helloworld)
target_link_libraries(test_version ${Boost_LIBRARIES} helloworld)

# Compiler warnings settings based on the compiler and platform
if (MSVC)
    target_compile_options(helloworld_cli PRIVATE /W4)
    target_compile_options(helloworld PRIVATE /W4)
    target_compile_options(test_version PRIVATE /W4)
elseif (CMAKE_SYSTEM_NAME STREQUAL "Linux" OR CMAKE_SYSTEM_NAME STREQUAL "Darwin")
    target_compile_options(helloworld_cli PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(helloworld PRIVATE -Wall -Wextra -pedantic -Werror)
    target_compile_options(test_version PRIVATE -Wall -Wextra -pedantic -Werror)
endif()

# Installation rules
install(TARGETS helloworld_cli RUNTIME DESTINATION bin)

# Set CPack configuration for packaging
set(CPACK_GENERATOR "DEB") # Specify only DEB if PKG is not needed

set(CPACK_PACKAGE_NAME "helloworld") # Set your package name
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "Your Name <your.email@example.com>") # Set maintainer info
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION "A simple hello world application") # Set description
set(CPACK_DEBIAN_PACKAGE_DEPENDS "") # Specify dependencies if any

include(CPack)

# Enable testing
enable_testing()

# Add tests
add_test(test_version test_version)