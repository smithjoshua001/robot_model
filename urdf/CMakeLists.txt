cmake_minimum_required(VERSION 2.8.3)
project(urdf)
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
find_package(urdfdom REQUIRED)
message(STATUS "urdfdom_INCLUDE_DIRS: ${urdfdom_INCLUDE_DIRS}")
message(STATUS "urdfdom_LIBRARIES: ${urdfdom_LIBRARIES}")

find_package(PkgConfig REQUIRED)
pkg_search_module(TINYXML2 REQUIRED TinyXML2 tinyxml2)
ExtractLibraries(TinyXML_LIBRARIES ${TINYXML2_LIBRARY_DIRS} ${TINYXML2_LIBRARIES})
message(STATUS "TINYXML2 LIBS: ${TinyXML_LIBRARIES}")

find_package(PkgConfig)
pkg_check_modules(libpcrecpp libpcrecpp)

include_directories(SYSTEM ${Boost_INCLUDE_DIR})
link_directories(${Boost_LIBRARY_DIRS})

include_directories(
  include
  ${urdfdom_INCLUDE_DIRS}
  ${TINYXML_INCLUDE_DIRS}
  )

add_library(${PROJECT_NAME} SHARED src/model.cpp)
target_link_libraries(${PROJECT_NAME} ${TINYXML_LIBRARIES} ${urdfdom_LIBRARIES})

if(APPLE)
  set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "-undefined dynamic_lookup")
endif(APPLE)
install(TARGETS ${PROJECT_NAME}
  DESTINATION "lib"
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION "include/${PROJECT_NAME}"
  FILES_MATCHING PATTERN "*.h"
)

set(LIBRARY_NAME "${CMAKE_PROJECT_NAME}")

# Configuration installation

configure_file("${CMAKE_PROJECT_NAME}-config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}-config.cmake"
    @ONLY)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_PROJECT_NAME}-config.cmake"
    DESTINATION "share/${CMAKE_PROJECT_NAME}")
