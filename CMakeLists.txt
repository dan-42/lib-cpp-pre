cmake_minimum_required (VERSION 3.5.0)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.20.14.tar.gz"
    SHA1 "18cdbf5c47abdda437c73bf5437b7c3c65afe47c"
)

project (cpp-pre VERSION 1.3.8)


# Compile unit tests
option(UNIT_TESTS "Enable Unit Testing" OFF)

if (UNIT_TESTS)
  enable_testing()
endif()

##
## CONFIGURATION
##
set (CMAKE_CXX_STANDARD 14)
set(LIB_CPP_PRE_TARGET_NAME               ${PROJECT_NAME})
set(LIB_CPP_PRE_INCLUDE_BUILD_DIR          "${PROJECT_SOURCE_DIR}/pre/")
set(LIB_CPP_PRE_CONFIG_INSTALL_DIR        "lib/cmake/${PROJECT_NAME}")
set(LIB_CPP_PRE_INCLUDE_INSTALL_DIR       "include")
set(LIB_CPP_PRE_TARGETS_EXPORT_NAME       "${PROJECT_NAME}Targets")
set(LIB_CPP_PRE_CMAKE_CONFIG_TEMPLATE     "cmake/config.cmake.in")
set(LIB_CPP_PRE_CMAKE_CONFIG_DIR          "${CMAKE_CURRENT_BINARY_DIR}/cmake_config")
set(LIB_CPP_PRE_CMAKE_VERSION_CONFIG_FILE "${LIB_CPP_PRE_CMAKE_CONFIG_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(LIB_CPP_PRE_CMAKE_PROJECT_CONFIG_FILE "${LIB_CPP_PRE_CMAKE_CONFIG_DIR}/${PROJECT_NAME}Config.cmake")


##
## TARGET
## create target and add include path
##
add_library(${LIB_CPP_PRE_TARGET_NAME} INTERFACE)

##
## target Config
##
if(MSVC)
    target_compile_definitions(${LIB_CPP_PRE_TARGET_NAME} PRIVATE NOMINMAX=1 BOOST_ALL_NO_LIB=1=1)
endif()

##
## Dependancies
##
## Boost
set(Boost_USE_MULTITHREADED ON)
hunter_add_package(Boost)
find_package(Boost 1.66 REQUIRED)
include_directories ( ${Boost_INCLUDE_DIRS} )

## nlohmann-json
hunter_add_package(nlohmann_json)
find_package(nlohmann_json REQUIRED)

target_include_directories(
    ${LIB_CPP_PRE_TARGET_NAME}
    INTERFACE
    $<BUILD_INTERFACE:${LIB_CPP_PRE_INCLUDE_BUILD_DIR}>
    $<INSTALL_INTERFACE:include>
)

target_include_directories(
    ${LIB_CPP_PRE_TARGET_NAME}
    INTERFACE
    $<BUILD_INTERFACE:${Boost_INCLUDE_DIRS}>
)

target_link_libraries(${LIB_CPP_PRE_TARGET_NAME} INTERFACE nlohmann_json)


##
## INSTALL
## install header files, generate and install cmake config files for find_package()
##
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${LIB_CPP_PRE_CMAKE_VERSION_CONFIG_FILE} COMPATIBILITY SameMajorVersion
)
configure_package_config_file(
    ${LIB_CPP_PRE_CMAKE_CONFIG_TEMPLATE}
    ${LIB_CPP_PRE_CMAKE_PROJECT_CONFIG_FILE}
    INSTALL_DESTINATION ${LIB_CPP_PRE_CONFIG_INSTALL_DIR}
)

install(
    DIRECTORY ${LIB_CPP_PRE_INCLUDE_BUILD_DIR}
    DESTINATION ${LIB_CPP_PRE_INCLUDE_INSTALL_DIR}
)
install(
    FILES ${LIB_CPP_PRE_CMAKE_PROJECT_CONFIG_FILE} ${LIB_CPP_PRE_CMAKE_VERSION_CONFIG_FILE}
    DESTINATION ${LIB_CPP_PRE_CONFIG_INSTALL_DIR}
)

install(
    TARGETS ${LIB_CPP_PRE_TARGET_NAME}
    EXPORT ${LIB_CPP_PRE_TARGETS_EXPORT_NAME}
    INCLUDES DESTINATION ${LIB_CPP_PRE_INCLUDE_INSTALL_DIR}
)
install(
    EXPORT ${LIB_CPP_PRE_TARGETS_EXPORT_NAME}
    DESTINATION ${LIB_CPP_PRE_CONFIG_INSTALL_DIR}
)


#install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/pre DESTINATION include
#        FILES_MATCHING PATTERN "*.[ih]pp")


# Unit testing
if (UNIT_TESTS)
  add_subdirectory(test)
endif()
