cmake_minimum_required(VERSION 2.8.3)

project(kdl_parser)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

function(ExtractLibraries outputVar libraryHints)
	foreach(LIB ${ARGN})
		message(STATUS "LIB: ${LIB}")
		find_library(TMP ${LIB} HINTS ${libraryHints})
		set( ${outputVar} ${${outputVar}} ${TMP})
		unset(TMP CACHE)
	endforeach(LIB)
  set(${outputVar} ${${outputVar}} PARENT_SCOPE)
endfunction()
if (NOT MSVC)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
endif()

find_package(orocos_kdl REQUIRED)

find_package(urdf REQUIRED)
include_directories(SYSTEM ${urdf_INCLUDE_DIRS})
message(STATUS "urdf_INCLUDE_DIRS: ${urdf_INCLUDE_DIRS}")
link_directories(SYSTEM ${urdf_LIBRARY_DIRS})
message(STATUS "urdf_LIBRARY_DIRS: ${urdf_LIBRARY_DIRS}")
message(STATUS "urdf_LIBRARIES: ${urdf_LIBRARIES}")

find_package(PkgConfig REQUIRED)
pkg_search_module(TINYXML2 REQUIRED TinyXML2 tinyxml2)
ExtractLibraries(TinyXML_LIBRARIES ${TINYXML2_LIBRARY_DIRS} ${TINYXML2_LIBRARIES})
message(STATUS "TINYXML2 LIBS: ${TinyXML_LIBRARIES}")

include_directories(include ${orocos_kdl_INCLUDE_DIRS} ${TINYXML2_INCLUDE_DIRS})

link_directories(${orocos_kdl_LIBRARY_DIRS})

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
