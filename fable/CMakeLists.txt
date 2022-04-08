cmake_minimum_required(VERSION 3.7 FATAL_ERROR)

project(fable LANGUAGES CXX)

set(target fable)
set(alias fable::fable)

find_package(Boost REQUIRED)
find_package(fmt REQUIRED)
find_package(nlohmann_json REQUIRED)

# Library ------------------------------------------------------------
add_library(${target}
    # find src -type f -name "*.cpp" \! -name "*_test.cpp"
    src/fable/conf.cpp
    src/fable/json.cpp
    src/fable/schema.cpp
    src/fable/schema/struct.cpp
    src/fable/schema/string.cpp
    src/fable/schema/variant.cpp
    src/fable/schema/path.cpp
    src/fable/environment.cpp
    src/fable/utility.cpp
)
add_library(${alias} ALIAS ${target})
set_target_properties(${target} PROPERTIES
    CXX_STANDARD 14
    CXX_STANDARD_REQUIRED ON
    VERSION ${FABLE_VERSION}
)
target_include_directories(${target}
  PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
    "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
)
target_link_libraries(${target}
  PUBLIC
    boost::boost
    fmt::fmt
    nlohmann_json::nlohmann_json
)

# Testing ------------------------------------------------------------
option(BuildTests "Build tests?" ON)
if(BuildTests)
    message(STATUS "-> Enable testing")
    enable_testing()
    find_package(GTest REQUIRED)
    include(GoogleTest)

    add_executable(test-fable
        # find src -type f -name "*_test.cpp"
        src/fable/environment_test.cpp
        src/fable/schema/const_test.cpp
        src/fable/schema/custom_test.cpp
        src/fable/schema/enum_test.cpp
        src/fable/schema/factory_test.cpp
        src/fable/schema/factory_advanced_test.cpp
        src/fable/schema/number_test.cpp
        src/fable/schema/optional_test.cpp
        src/fable/schema/struct_test.cpp
        src/fable/schema_test.cpp
    )
    set_target_properties(test-fable PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
    )
    target_link_libraries(test-fable
        gtest::gtest
        boost::boost
        ${target}
    )
    gtest_add_tests(TARGET test-fable)
endif()

option(BuildExamples "Build examples?" ${BuildTests})
if(BuildExamples)
message(STATUS "-> Build examples")
    find_package(cli11 REQUIRED)
    add_executable(example-contacts
        examples/contacts/src/main.cpp
    )
    set_target_properties(example-contacts PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
    )
    target_link_libraries(example-contacts
        cli11::cli11
        ${target}
    )

    add_executable(example-simple-config
        examples/simple_config/src/main.cpp
    )
    set_target_properties(example-simple-config PROPERTIES
        CXX_STANDARD 14
        CXX_STANDARD_REQUIRED ON
    )
    target_link_libraries(example-simple-config
        cli11::cli11
        ${target}
    )
endif()

# Installation -------------------------------------------------------
include(GNUInstallDirs)

# install the target related to export-set
install(TARGETS fable EXPORT fableTargets)

install(
    DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Install all exported targets
install(EXPORT fableTargets
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/fable
    NAMESPACE fable::
)

include(CMakePackageConfigHelpers)
# create config file
configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cfg/pkg/Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/fableConfig.cmake"
    INSTALL_DESTINATION cmake
)

# install config files
install(
    FILES
        "${CMAKE_CURRENT_BINARY_DIR}/fableConfig.cmake"
    DESTINATION lib/cmake/fable
    COMPONENT "cmake"
)
