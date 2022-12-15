find_package(Doxygen)

if(NOT DOXYGEN_FOUND)
	message(WARNING "Doxygen is not found so documentation will not be build")
	return()
endif()

set(SOIL_DOCS_INPUT_DIRS "${PROJECT_SOURCE_DIR}/include/SOIL/SOIL.h ${PROJECT_BINARY_DIR}/include")
set(SOIL_DOCS_OUTPUT_DIR "${PROJECT_BINARY_DIR}/docs")
set(SOIL_DOCS_ARTIFACT "${SOIL_DOCS_OUTPUT_DIR}/html/index.html")
set(SOIL_DOCS_DOXYFILE "${PROJECT_BINARY_DIR}/Doxyfile")
configure_file("${CMAKE_CURRENT_LIST_DIR}/Doxyfile.in" "${SOIL_DOCS_DOXYFILE}")
add_custom_command(OUTPUT ${SOIL_DOCS_ARTIFACT}
	COMMAND ${DOXYGEN_EXECUTABLE} ${SOIL_DOCS_DOXYFILE}
	COMMENT "Generating Doxygen documentation"
	VERBATIM
)
add_custom_target(docs ALL DEPENDS ${SOIL_DOCS_ARTIFACT})

list(APPEND ADDITIONAL_CLEAN_FILES "${PROJECT_BINARY_DIR}/docs/html/")

if(NOT SOIL_BUNDLED)
	install(
		DIRECTORY "${PROJECT_BINARY_DIR}/docs/html"
		DESTINATION "${CMAKE_INSTALL_DATADIR}/SOIL/docs"
	)
endif()
