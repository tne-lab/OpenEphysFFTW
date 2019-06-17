cmake_minimum_required(VERSION 3.5.0)
if (NOT DEFINED GUI_BASE_DIR)
	if (DEFINED ENV{GUI_BASE_DIR})
		set(GUI_BASE_DIR $ENV{GUI_BASE_DIR})
	else()
		set(GUI_BASE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../../plugin-GUI)
	endif()
endif()

get_filename_component(PROJECT_FOLDER ${CMAKE_CURRENT_SOURCE_DIR}/.. ABSOLUTE)
get_filename_component(COMMONLIB_NAME ${PROJECT_FOLDER} NAME)

project(OE_COMMONLIB_${COMMONLIB_NAME})

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
	set(LINUX 1)
endif()

set(SOURCE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/../Source)
file(GLOB_RECURSE SRC_FILES LIST_DIRECTORIES false "${SOURCE_PATH}/*.cpp" "${SOURCE_PATH}/*.h")

add_library(${COMMONLIB_NAME} SHARED ${SRC_FILES})

target_compile_definitions(${COMMONLIB_NAME} PUBLIC
	"$<$<PLATFORM_ID:Windows>:JUCE_API=__declspec(dllimport)>"
	$<$<PLATFORM_ID:Windows>:_CRT_SECURE_NO_WARNINGS>
	$<$<PLATFORM_ID:Linux>:JUCE_DISABLE_NATIVE_FILECHOOSERS=1>
	$<$<CONFIG:Debug>:DEBUG=1>
	$<$<CONFIG:Debug>:_DEBUG=1>
	$<$<CONFIG:Release>:NDEBUG=1>
	)

target_compile_features(${COMMONLIB_NAME} PUBLIC cxx_auto_type cxx_generalized_initializers)
target_include_directories(${COMMONLIB_NAME} PUBLIC ${GUI_BASE_DIR}/JuceLibraryCode ${GUI_BASE_DIR}/JuceLibraryCode/modules ${GUI_BASE_DIR}/Source/Plugins/Headers)

#Libraries and compiler options
if(MSVC)
	set(GUI_BIN_DIR ${GUI_BASE_DIR}/Builds/VisualStudio2013)

	if (CMAKE_GENERATOR_PLATFORM STREQUAL "x64")
		set(GUI_BIN_DIR ${GUI_BIN_DIR}/x64)
		set(CMAKE_LIBRARY_ARCHITECTURE "x64")
	elseif(CMAKE_GENERATOR_PLATFORM STREQUAL "Win32")
		set(CMAKE_LIBRARY_ARCHITECTURE "x86")
	else()
		message(FATAL_ERROR "Unsupported generator platform ${CMAKE_GENERATOR_PLATFORM}")
	endif()

	set(GUI_BIN_DIR ${GUI_BIN_DIR}/$<IF:$<CONFIG:Debug>,Debug,Release>/bin)

	target_link_libraries(${COMMONLIB_NAME} ${GUI_BIN_DIR}/open-ephys.lib)
	target_compile_options(${COMMONLIB_NAME} PRIVATE /sdl-)

	install(TARGETS ${COMMONLIB_NAME}
		RUNTIME DESTINATION ${GUI_BIN_DIR}/shared
		ARCHIVE DESTINATION ${GUI_BIN_DIR}/lib)

	set(WINLIBS_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../../libs)

	# also install required DLLs
	install(DIRECTORY ${WINLIBS_DIR}/bin/${CMAKE_LIBRARY_ARCHITECTURE}/
		DESTINATION ${GUI_BIN_DIR}/shared)

	set(CMAKE_PREFIX_PATH ${WINLIBS_DIR})

elseif(LINUX)
	set(GUI_BIN_DIR ${GUI_BASE_DIR}/Builds/Linux/build)

	target_link_libraries(${COMMONLIB_NAME} GL X11 Xext Xinerama asound dl freetype pthread rt)
	set_property(TARGET ${COMMONLIB_NAME} APPEND_STRING PROPERTY LINK_FLAGS
		"-fvisibility=hidden -fPIC -rdynamic -Wl,-rpath,'$ORIGIN'")
	target_compile_options(${COMMONLIB_NAME} PRIVATE -fPIC -rdynamic)
	target_compile_options(${COMMONLIB_NAME} PRIVATE -O3) #enable optimization for linux debug
	
	install(TARGETS ${COMMONLIB_NAME} LIBRARY DESTINATION ${GUI_BIN_DIR}/shared)
	set(CMAKE_LIBRARY_PATH ${GUI_BIN_DIR}/shared)

elseif(APPLE)
	set(GUI_BIN_DIR $ENV{HOME}/Library/Application\ Support/open-ephys)
	set_target_properties(${COMMONLIB_NAME} PROPERTIES BUNDLE FALSE)
	set_property(TARGET ${COMMONLIB_NAME} APPEND_STRING PROPERTY LINK_FLAGS
	"-undefined dynamic_lookup")

	install(TARGETS ${COMMONLIB_NAME} DESTINATION ${GUI_BIN_DIR}/shared)

	set(CMAKE_LIBRARY_PATH ${GUI_BIN_DIR}/shared)
	set(CMAKE_PREFIX_PATH /opt/local)
endif()

#copy and include common lib header files

install(FILES ${SOURCE_PATH}/OpenEphysFFTW.h DESTINATION ${GUI_BIN_DIR}/include)
target_include_directories(${COMMONLIB_NAME} PUBLIC ${GUI_BIN_DIR}/include)

#create filters for vs and xcode

foreach( src_file IN ITEMS ${SRC_FILES})
	get_filename_component(src_path "${src_file}" PATH)
	file(RELATIVE_PATH src_path_rel "${SOURCE_PATH}" "${src_path}")
	string(REPLACE "/" "\\" group_name "${src_path_rel}")
	source_group("${group_name}" FILES "${src_file}")
endforeach()

#additional libraries

find_library(FFTW_LIBRARY NAMES libfftw3-3 fftw3 fftw3-3)
find_path(FFTW_INCLUDE_DIRS fftw3.h)

if (NOT FFTW_LIBRARY)
	message(FATAL_ERROR "FFTW library not found")
endif()

if (NOT FFTW_INCLUDE_DIRS)
	message(FATAL_ERROR "FFTW header not found")
endif()

target_link_libraries(${COMMONLIB_NAME} ${FFTW_LIBRARY})
target_include_directories(${COMMONLIB_NAME} PRIVATE ${FFTW_INCLUDE_DIRS})
