# Project information.
cmake_minimum_required( VERSION 3.5.0 )
project( AwsIotDeviceSdkC
         VERSION 4.0.0
         LANGUAGES C )
add_definitions( -DAWS_IOT_SDK_VERSION="${PROJECT_VERSION}" )

# Use C99.
set( CMAKE_C_STANDARD 99 )
set( CMAKE_C_STANDARD_REQUIRED ON )

# Do not allow in-source build.
if( ${PROJECT_SOURCE_DIR} STREQUAL ${PROJECT_BINARY_DIR} )
    message( FATAL_ERROR "In-source build is not allowed. Please build in a separate directory, such as ${PROJECT_SOURCE_DIR}/build." )
endif()

# Check for system support.
list( APPEND SUPPORTED_SYSTEMS "Linux" )

if( NOT ${CMAKE_SYSTEM_NAME} IN_LIST SUPPORTED_SYSTEMS )
    message( FATAL_ERROR "Unsupported system: ${CMAKE_SYSTEM_NAME}. Currently supported systems are: ${SUPPORTED_SYSTEMS}." )
endif()

# Set output directories.
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib )

# SDK include path.
include_directories( ${PROJECT_SOURCE_DIR}/lib/include )

# CMake configure_file output.
include_directories( ${PROJECT_BINARY_DIR}/config )

# Demo include path.
include_directories( ${PROJECT_SOURCE_DIR}/demos )

# Use either the demo or test configuration file.
if( ${AWS_IOT_BUILD_TESTS} )
    # Tests include directories.
    include_directories( ${PROJECT_SOURCE_DIR}/tests
                         tests/unity
                         tests/unity/fixture
                         tests/mqtt/access )

    # Tests config file.
    add_definitions( -DAWS_IOT_USER_CONFIG_FILE="aws_iot_tests_config.h" )

    # Build unity test framework.
    add_subdirectory( tests/unity )
else()
    add_definitions( -DAWS_IOT_USER_CONFIG_FILE="aws_iot_demo_config.h" )
endif()

# Platform libraries.
if( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" )
    add_subdirectory( lib/source/platform/posix )

    # Set the configuration file for all subsequent libraries.
    add_definitions( -DAWS_IOT_CONFIG_FILE="aws_iot_config_build.h" )
endif()

# Common libraries (queue, logging, etc.)
add_subdirectory( lib/source/common )

# MQTT library.
add_subdirectory( lib/source/mqtt )

# Shadow library.
add_subdirectory( lib/source/shadow )

# Demo executables.
if( ${CMAKE_SYSTEM_NAME} STREQUAL "Linux" )
    add_subdirectory( demos/posix )
endif()

# Test executables.
if( ${AWS_IOT_BUILD_TESTS} )
    add_subdirectory( tests/ )
endif()