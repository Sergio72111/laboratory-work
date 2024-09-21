cmake_minimum_required(VERSION 3.10)

set(PATCH_VERSION "1" CACHE INTERNAL "Patch version")
set(PROJECT_VERSION 0.0.${PATCH_VERSION})

project(helloworld VERSION ${PROJECT_VERSION})

find_package(Boost COMPONENTS unit_test_framework REQUIRED)

configure_file(version.h.in version.h)

add_library(helloworld lib.cpp)  # Create a library from lib.cpp
add_library(helloworld_helper helper.cpp)  # Create another library from helper.cpp

add_executable(helloworld_cli main.cpp)  # Create an executable from main.cpp
target_link_libraries(helloworld_cli helloworld helloworld_helper)  # Link the executable with both libraries

set_target_properties(helloworld_cli PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
)

target_include_directories(helloworld PRIVATE "${CMAKE_BINARY_DIR}")
target_include_directories(helloworld_helper PRIVATE "${CMAKE_BINARY_DIR}")

set_target_properties(test_version PROPERTIES
    COMPILE_DEFINITIONS BOOST_TEST_DYN_LINK
    INCLUDE_DIRECTORIES ${Boost_INCLUDE_DIR}
)

target_link_libraries(test_version ${Boost_LIBRARIES} helloworld helloworld_helper)

if (APPLE)
    set(CPACK_GENERATOR "Bundle")  # For creating .app or .pkg on macOS
else()
    set(CPACK_GENERATOR "DEB")  # For creating .deb on Linux
endif()

set(CPACK_BUNDLE_NAME "LABABAB")  # Replace with your app name
set(CPACK_BUNDLE_PLIST "${CMAKE_CURRENT_SOURCE_DIR}/resources/Info.plist")
set(CPACK_BUNDLE_ICON "${CMAKE_CURRENT_SOURCE_DIR}/resources/ic.icns")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_CONTACT "example@example.com")

include(CPack)

enable_testing()
add_test(test_version test_version)