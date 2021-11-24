cmake_minimum_required( VERSION 3.20 )
project( CMake-ImGui-SFML )

set( CMAKE_CXX_STANDARD 23 )

set( CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g3 -Wall -Wextra" )
set( CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3" )

set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib )
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib )

set( CIF_DEPS ${CMAKE_SOURCE_DIR}/_deps )

# Include OpenGL
find_package( OpenGL REQUIRED )

# Include SFML
add_subdirectory( ${CIF_DEPS}/SFML )
include_directories( ${CIF_DEPS}/SFML/include )
set( CIF_SFML_LIBS sfml-system sfml-window sfml-network sfml-graphics sfml-audio )

# Include ImGui
set( CIF_IMGUI_DIR ${CIF_DEPS}/imgui )
set( CIF_IMGUI_SFML_DIR ${CIF_DEPS}/imgui-sfml )

set( CIF_IMGUI_FILES
        ${CIF_IMGUI_DIR}/imconfig.h
        ${CIF_IMGUI_DIR}/imgui.cpp
        ${CIF_IMGUI_DIR}/imgui.h
        ${CIF_IMGUI_DIR}/imgui_demo.cpp
        ${CIF_IMGUI_DIR}/imgui_draw.cpp
        ${CIF_IMGUI_DIR}/imgui_internal.h
        ${CIF_IMGUI_DIR}/imgui_tables.cpp
        ${CIF_IMGUI_DIR}/imgui_widgets.cpp
        ${CIF_IMGUI_DIR}/imstb_rectpack.h
        ${CIF_IMGUI_DIR}/imstb_textedit.h
        ${CIF_IMGUI_DIR}/imstb_truetype.h
        ${CIF_IMGUI_DIR}/misc/cpp/imgui_stdlib.cpp
        ${CIF_IMGUI_DIR}/misc/cpp/imgui_stdlib.h
        ${CIF_IMGUI_DIR}/misc/fonts/binary_to_compressed_c.cpp
        ${CIF_IMGUI_SFML_DIR}/imconfig-SFML.h
        ${CIF_IMGUI_SFML_DIR}/imgui-SFML.h
        ${CIF_IMGUI_SFML_DIR}/imgui-SFML.cpp
        ${CIF_IMGUI_SFML_DIR}/imgui-SFML_export.h
        )

include_directories( ${CIF_IMGUI_DIR} )
include_directories( ${CIF_IMGUI_SFML_DIR} )

add_library( ImGui-SFML SHARED ${CIF_IMGUI_FILES} )
target_link_libraries( ImGui-SFML ${CIF_SFML_LIBS} ${OPENGL_LIBRARIES} )

foreach( file ${CIF_IMGUI_FILES} )
	set( destination "" )
	get_filename_component( dir ${file} DIRECTORY )
	while( ${dir} STRNEQUAL ${CIF_IMGUI_DIR} AND ${dir} STRNEQUAL ${CIF_IMGUI_SFML_DIR} )
		get_filename_component( name ${dir} NAME )
		set( destination "${name}/${destination}" )
		get_filename_component( dir ${dir} DIRECTORY )
	endwhile()

	add_custom_command(
		TARGET ImGui-SFML POST-BUILD
		COMMAND ${CMAKE_COMMAND} -E
		copy ${file} "${CMAKE_SOURCE_DIR}/include/${destination}"
	)
endforeach()

