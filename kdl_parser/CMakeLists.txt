cmake_minimum_required(VERSION 2.8.3)

project(kdl_parser)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

find_package(Boost REQUIRED)
include_directories(${Boost_INCLUDE_DIR})

find_package(orocos_kdl REQUIRED)

find_package(urdf REQUIRED)
include_directories(SYSTEM ${urdf_INCLUDE_DIRS})
message(STATUS "urdf_INCLUDE_DIRS: ${urdf_INCLUDE_DIRS}")
link_directories(SYSTEM ${urdf_LIBRARY_DIRS})
message(STATUS "urdf_LIBRARY_DIRS: ${urdf_LIBRARY_DIRS}")
message(STATUS "urdf_LIBRARIES: ${urdf_LIBRARIES}")

find_package(PkgConfig REQUIRED)
pkg_search_module(TINYXML REQUIRED tinyxml)
if(NOT TINYXML_INCLUDE_DIR)
        find_path("/usr/include" tinyxml.h)
        find_library(TINYXML_LIBRARY NAMES tinyxml)
        set(TINYXML_LIBRARIES ${TINYXML_LIBRARY})
    set(TINYXML_INCLUDE_DIRS ${TINYXML_INCLUDE_DIR})
    set(TINYXML_LIBRARY_DIR "/usr/lib/x86_64-linux-gnu")
endif()
message(STATUS "TINYXML_VERSION: ${TINYXML_VERSION}")
include_directories(BEFORE SYSTEM ${TINYXML_INCLUDE_DIRS})
link_directories(${TINYXML_LIBRARY_DIR})
message(STATUS "TINYXML_LIBRARIES: ${TINYXML_LIBRARIES}")
message(STATUS "TINYXML_LIBRARY_DIR: ${TINYXML_LIBRARY_DIR}")
message(STATUS "TINYXML_INCLUDE_DIR: ${TINYXML_INCLUDE_DIRS}")
message(STATUS "TINYXML_LIBRARY: ${TINYXML_LIBRARY}")

include_directories(include ${orocos_kdl_INCLUDE_DIRS} ${TINYXML_INCLUDE_DIRS})

link_directories(${orocos_kdl_LIBRARY_DIRS})
link_directories(${Boost_LIBRARY_DIRS})

find_library(KDL_LIBRARY REQUIRED NAMES orocos-kdl HINTS ${orocos_kdl_LIBRARY_DIRS})

add_library(${PROJECT_NAME} SHARED src/kdl_parser.cpp)
target_link_libraries(${PROJECT_NAME}
  ${TINYXML_LIBRARIES} ${orocos_kdl_LIBRARIES} ${urdf_LIBRARIES}
)

install(TARGETS ${PROJECT_NAME}
  DESTINATION "lib"
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION "include/${PROJECT_NAME}"
  FILES_MATCHING PATTERN "*.hpp"
)

set(LIBRARY_NAME "${CMAKE_PROJECT_NAME}")
# Configuration installation

configure_file("${CMAKE_PROJECT_NAME}-config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}-config.cmake"
    @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}-config.cmake"
    DESTINATION "share/${CMAKE_PROJECT_NAME}")
