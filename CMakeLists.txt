cmake_minimum_required(VERSION 2.8)
project(sscanf)

set(SSCANF_PLUGIN_VERSION "3.0-beta1")

list(APPEND CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/libs/cmake")

# if(NOT COMPILER_SUPPORTS_CXX11)
	# message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} does not fulfill all required standards. Please use a different C++ compiler.")
# endif()


add_subdirectory(src)
