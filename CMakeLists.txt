cmake_minimum_required(VERSION 2.8.0)
project(ReverseTale-Library)

# ---[ Includes
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake-utils)

# -[ Macros and functions
include(Configure)
include(Macros)
include(External)
ResetAllTargets()

# ---[ Create project
CreateTarget(TARGET ReverseTale-Library)

set(SOURCE_FOLDERS
	${CMAKE_CURRENT_SOURCE_DIR}/src
	${CMAKE_CURRENT_SOURCE_DIR}/src/Cryptography
	${CMAKE_CURRENT_SOURCE_DIR}/src/Tools
	${CMAKE_CURRENT_SOURCE_DIR}/src/Game
)

foreach(folder ${SOURCE_FOLDERS})
	get_filename_component(FOLDER_NAME ${folder} NAME)

	AddToSources(
		TARGET ReverseTale-Library
		SRC_PATH ${folder}
		GLOB_SEARCH ".hpp;.cpp"
		FOLDER_NAME ${FOLDER_NAME}
	)
endforeach()

# ---[ Load dependencies
add_subdirectory(dep)

ResolveExternal(TARGET ReverseTale-Library)
if (ReverseTale-Library_IS_RESOLVED)
	# ---[ Setup include directories
	AddToIncludes(
		TARGET ReverseTale-Library
		INC_PATH ${CMAKE_CURRENT_SOURCE_DIR}/include
	)
	# AddToIncludes(
	#     TARGET ReverseTale-Library
	#     INC_PATH ${PROJECT_BINARY_DIR}
	# )


	# ---[ Setup some definitions
	set(DEFINITIONS "")
	if (NOT ${DEBUG})
		set(DEFINITIONS "${DEFINITIONS} NDEBUG=1")
	endif()

	AddLibrary(
		TARGET ReverseTale-Library
		LIBRARY ws2_32
		PLATFORM WIN32
	)

	BuildNow(
		TARGET ReverseTale-Library
		DEFINES ${DEFINITIONS}
		STATIC_LIB
		OUTPUT_NAME Library
		C++11
	)

	MakeInstallable(TARGET ReverseTale-Library)
endif()
