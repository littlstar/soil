cmake_minimum_required(VERSION 3.9)
project(SOIL
	VERSION 0.1.1
	LANGUAGES C
)
set(SOIL_VERSION ${PROJECT_VERSION})

if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type" FORCE)
    set_property(CACHE CMAKE_BUILD_TYPE
        PROPERTY STRINGS "Release" "Debug" "RelWithDebInfo" "MinSizeRel"
    )
endif()
if (NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "SOIL build type: ${CMAKE_BUILD_TYPE}")
endif()

if(NOT DEFINED SOIL_BUNDLED)
	if("${PROJECT_SOURCE_DIR}" STREQUAL "${CMAKE_SOURCE_DIR}")
		set(SOIL_BUNDLED FALSE)
	else()
		set(SOIL_BUNDLED TRUE)
	endif()
endif()
set(SOIL_BUNDLED ${SOIL_BUNDLED} CACHE BOOL "" FORCE)
mark_as_advanced(SOIL_BUNDLED)

include(CMakeDependentOption)
CMAKE_DEPENDENT_OPTION(BUILD_SHARED_LIBS "Build SOIL as shared library"
    OFF
    "NOT SOIL_BUNDLED"
    OFF
)

if(NOT SOIL_BUNDLED)
	include(GNUInstallDirs)
	set(SOIL_TARGETS "SOILTargets")
	set(SOIL_BUILD_TEST_APP_DEFAULT TRUE)
	set(SOIL_BUILD_BENCHMARKS_DEFAULT FALSE)
else()
	set(SOIL_BUILD_TEST_APP_DEFAULT FALSE)
	set(SOIL_BUILD_BENCHMARKS_DEFAULT FALSE)
endif()

add_library(libsoil)
add_library(${PROJECT_NAME}::libsoil ALIAS libsoil)
set_target_properties(libsoil
    PROPERTIES
    OUTPUT_NAME soil
)
if(BUILD_SHARED_LIBS)
    set_target_properties(libsoil
        PROPERTIES
        SOVERSION "${PROJECT_VERSION_MAJOR}"
        VERSION "${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}"
    )
endif()

# configure SOIL_defs.h {{
option(SOIL_CHECK_FOR_GL_ERRORS "Should SOIL check OpenGL errors or not" FALSE)
set(SOIL_CHECK_FOR_GL_ERRORS_DEFINE 0)
if (SOIL_CHECK_FOR_GL_ERRORS)
    set(SOIL_CHECK_FOR_GL_ERRORS_DEFINE 1)
endif()

get_target_property(SOIL_NAME SOIL::libsoil OUTPUT_NAME)
if (NOT ${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    set(SOIL_NAME "lib${SOIL_NAME}")
endif()
set(SOIL_LINKAGE "static")
if(BUILD_SHARED_LIBS)
    set(SOIL_LINKAGE "shared")
endif()
set(SOIL_DATE_COMPILED)
string(TIMESTAMP SOIL_DATE_COMPILED UTC)
set(SOIL_VERSION_REVISION ${PROJECT_VERSION_TWEAK})
if("${SOIL_VERSION_REVISION}" STREQUAL "")
    set(SOIL_VERSION_REVISION 0)
endif()
set(SOIL_VERSION_COMBINED)
math(EXPR SOIL_VERSION_COMBINED
    "${PROJECT_VERSION_MAJOR} << 24 | \
     ${PROJECT_VERSION_MINOR} << 16 | \
     ${PROJECT_VERSION_PATCH} << 8 | \
     ${SOIL_VERSION_REVISION}"
    OUTPUT_FORMAT DECIMAL
)
mark_as_advanced(
  SOIL_CHECK_FOR_GL_ERRORS_DEFINE
  SOIL_NAME
  SOIL_LINKAGE
  SOIL_DATE_COMPILED
  SOIL_VERSION_REVISION
  SOIL_VERSION_COMBINED
)
configure_file(
    "${CMAKE_CURRENT_LIST_DIR}/cmake/SOIL_defs.h.in"
    "${PROJECT_BINARY_DIR}/include/SOIL_defs.h"
    NEWLINE_STYLE UNIX
)
# }}

target_include_directories(libsoil
	PUBLIC
	$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}/include/>
    $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}/include/>
	$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/>
)

set(HEADERS
    include/SOIL/SOIL.h
    include/SOIL/image_dxt.h
    include/SOIL/image_helper.h
    include/SOIL/stb_image_aug.h
    include/SOIL/stbi_dds_aug.h
    include/SOIL/stbi_dds_aug_c.h
)
set(SOURCES
    src/soil.c
    src/image_dxt.c
    src/image_helper.c
    src/stb_image_aug.c
)
source_group(TREE ${CMAKE_CURRENT_LIST_DIR}
	PREFIX ""
    FILES ${HEADERS} ${SOURCES}
)

target_sources(libsoil
    PRIVATE
    ${SOURCES}
    ${HEADERS}
)

if(NOT SOIL_BUNDLED)
    install(TARGETS libsoil
        EXPORT ${SOIL_TARGETS}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}/
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/
	)
    install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )
    install(DIRECTORY ${PROJECT_BINARY_DIR}/include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

	include(CMakePackageConfigHelpers)
	set(SOIL_VERSION_CONFIG ${PROJECT_BINARY_DIR}/SOILConfigVersion.cmake)
	set(SOIL_PROJECT_CONFIG ${PROJECT_BINARY_DIR}/SOILConfig.cmake)
	set(SOIL_TARGETS_CONFIG ${PROJECT_BINARY_DIR}/SOILTargets.cmake)
	set(SOIL_CONFIG_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/SOIL/)
	write_basic_package_version_file(
		${SOIL_VERSION_CONFIG}
		VERSION ${SOIL_VERSION}
		COMPATIBILITY SameMajorVersion
		ARCH_INDEPENDENT
	)
	configure_package_config_file(
		${CMAKE_CURRENT_LIST_DIR}/cmake/SOILConfig.cmake.in
		${SOIL_PROJECT_CONFIG}
		INSTALL_DESTINATION ${SOIL_CONFIG_INSTALL_DIR}
	)

	export(EXPORT ${SOIL_TARGETS}
		FILE ${SOIL_TARGETS_CONFIG}
		NAMESPACE SOIL::
	)
	install(EXPORT ${SOIL_TARGETS}
		DESTINATION ${SOIL_CONFIG_INSTALL_DIR}
		NAMESPACE SOIL::
	)
	install(
		FILES ${SOIL_PROJECT_CONFIG} ${SOIL_VERSION_CONFIG}
		DESTINATION ${SOIL_CONFIG_INSTALL_DIR}
	)
endif()
