cmake_minimum_required(VERSION 3.31)
project(curlpp CXX)

set(required_cxx_version 11)

if(NOT DEFINED CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD ${required_cxx_version})
elseif(CMAKE_CXX_STANDARD LESS ${required_cxx_version})
  message(FATAL_ERROR "cURLpp requires C++ ${required_cxx_version} or higher.")
endif()

if(BUILD_SHARED_LIBS)
  message(FATAL_ERROR "cURLpp requires BUILD_SHARED_LIBS to be off.")
endif()

if(NOT TARGET libcurl_static)
  message(FATAL_ERROR "cURLpp requires libcURL to be available before being integrated.")
endif()

include(FetchContent)
FetchContent_Declare(fetch_github_content GIT_REPOSITORY https://github.com/CesarBerriot/fetch_github_content.git GIT_TAG 1.0.0)
FetchContent_MakeAvailable(fetch_github_content)
include(${fetch_github_content_SOURCE_DIR}/fetch_github_content.cmake)

fetch_github_content(discover_sources CesarBerriot 1.0.0 INCLUDE_DEFAULT)

add_library(curlpp)
discover_sources(curlpp)
target_include_directories(curlpp PUBLIC include)
target_link_libraries(curlpp PUBLIC libcurl_static)
if(MINGW)
  target_compile_options(curlpp PUBLIC -Wno-deprecated-declarations)
endif()

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
  file(GLOB glob CONFIGURE_DEPENDS examples/*.cpp)
  foreach(entry IN LISTS glob)
    get_filename_component(file_name ${entry} NAME_WLE)
    set(name curlpp_${file_name})
    add_executable(${name} ${entry})
    target_link_libraries(${name} PRIVATE curlpp)
  endforeach()
endif()