cmake_minimum_required(VERSION 3.3)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.19.51.tar.gz"
    SHA1 "d238dc1dd4db83e45a592f96fdb95d17c688600a"
)

set(CURLCPP_VERSION_MAJOR 1)
set(CURLCPP_VERSION_MINOR 0)
set(CURLCPP_VERSION_PATCH 0)
set(CURLCPP_VERSION ${CURLCPP_VERSION_MAJOR}.${CURLCPP_VERSION_MINOR}.${CURLCPP_VERSION_PATCH})

project(CURLCPP VERSION ${CURLCPP_VERSION})

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")

option(BUILD_TESTS "Build tests" OFF)

hunter_add_package(CURL)
find_package(CURL CONFIG REQUIRED)

file(GLOB ${PROJECT_NAME}_SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/sources/*.cpp")
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include/)

add_library(libcurlcpp ${${PROJECT_NAME}_SOURCES})
set_target_properties(libcurlcpp PROPERTIES PREFIX "")
set_target_properties(libcurlcpp PROPERTIES IMPORT_PREFIX "")

target_link_libraries(libcurlcpp CURL::libcurl)

target_include_directories(libcurlcpp PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
# Install

set(GENERATED_DIR "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(CONFIG_INSTALL_DIR "lib/cmake/${PROJECT_NAME}")
set(INCLUDE_INSTALL_DIR "include")

set(VERSION_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
set(PROJECT_CONFIG "${GENERATED_DIR}/${PROJECT_NAME}Config.cmake")
set(TARGETS_EXPORT_NAME "${PROJECT_NAME}Targets")
set(NAMESPACE "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${VERSION_CONFIG}" COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "${PROJECT_SOURCE_DIR}/cmake/Config.cmake.in"
    "${PROJECT_CONFIG}"
     INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
)

install(TARGETS libcurlcpp
    EXPORT "${TARGETS_EXPORT_NAME}"
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include)

install(
  FILES "${PROJECT_CONFIG}" "${VERSION_CONFIG}"
  DESTINATION "${CONFIG_INSTALL_DIR}"
  )

install(EXPORT "${TARGETS_EXPORT_NAME}" 
    NAMESPACE "${NAMESPACE}" 
    DESTINATION "${CONFIG_INSTALL_DIR}"
)

if(BUILD_TESTS)
	enable_testing()
	file(GLOB ${PROJECT_NAME}_TEST_SOURCES tests/*.cpp)
	add_executable(check ${${PROJECT_NAME}_TEST_SOURCES})
	target_link_libraries(check libcurlcpp)
	add_test(NAME check COMMAND check) 
endif()
