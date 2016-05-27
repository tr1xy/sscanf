include(AMXConfig)
include(AddSAMPPlugin)


set(SAMP_SDK_ROOT "${PROJECT_SOURCE_DIR}/libs/sdk")
find_package(SAMPSDK REQUIRED)

set(SSCANF_CONFIG_INCLUDE_DIR "${CMAKE_CURRENT_BINARY_DIR}/config_headers")
include_directories(
	${SAMPSDK_INCLUDE_DIR}
	${SSCANF_CONFIG_INCLUDE_DIR}
)

configure_file(
	"../sscanf3.inc.in"
	"sscanf3.inc"
	@ONLY
)

configure_file(
	"version.hpp.in"
	"${SSCANF_CONFIG_INCLUDE_DIR}/version.hpp"
	@ONLY
)

add_samp_plugin(sscanf3 
	${SAMPSDK_DIR}/amxplugin.cpp
	${SAMPSDK_DIR}/amxplugin2.cpp
	${SAMPSDK_DIR}/amx/getch.c
	delimiters.cpp
	delimiters.hpp
	environment.cpp
	environment.hpp
	environment_skip.hpp
	errors.cpp
	errors.hpp
	memory.cpp
	memory.hpp
	options.cpp
	options.hpp
	parser.cpp
	parser.hpp
	players.cpp
	specifiers.cpp
	specifiers.hpp
	sscanf.cpp
	sscanf.hpp
	utils.cpp
	utils.hpp
	memories/default_memory.hpp
	specifiers/array_specifier.hpp
	specifiers/delim_specifier.hpp
	specifiers/enum_specifier.hpp
	specifiers/group_specifiers.cpp
	specifiers/group_specifiers.hpp
	specifiers/numeric_specifier.hpp
	specifiers/other_specifiers.cpp
	specifiers/other_specifiers.hpp
	specifiers/packed_specifier.hpp
	specifiers/simple_specifiers.cpp
	specifiers/simple_specifiers.hpp
	specifiers/specnum_specifiers.hpp
	specifiers/string_specifier.cpp
	specifiers/string_specifier.hpp
	specifiers/trivial_specifiers.hpp
	${SSCANF_CONFIG_INCLUDE_DIR}/version.hpp
	sscanf.def
)

#add_dependencies(sscanf3 log-core cppformat)

if(MSVC)
	add_definitions(-D_CRT_SECURE_NO_WARNINGS -DNOMINMAX)
endif()

#target_link_libraries(sscanf3 log-core cppformat)

if(UNIX)
	set_target_properties(sscanf3 PROPERTIES
		INSTALL_RPATH "$ORIGIN/..:$ORIGIN/")
endif()

install(TARGETS sscanf3 DESTINATION "plugins/")
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/sscanf3.inc" DESTINATION "pawno/include/")

set(CPACK_PACKAGE_VERSION ${SSCANF_PLUGIN_VERSION})
if(WIN32)
	set(CPACK_GENERATOR ZIP)
else()
	set(CPACK_GENERATOR TGZ)
endif()

include(CPack)
