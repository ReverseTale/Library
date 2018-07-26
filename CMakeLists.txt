cmake_minimum_required(VERSION 2.8.0)

# ---[ Includes
set(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake-utils)

# -[ Macros and functions
include(Configure)
include(Macros)
include(External)
ResetAllTargets()

# ---[ Create project
project(ReverseTale-Library)
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
find_package(fmt REQUIRED)

if (USE_SYSTEM_BOOST)
    find_package(Boost 1.62 REQUIRED)
    set(BOOST_DEPENDENCIES "")

    AddDependency(
        TARGET ReverseTale-Library
        DEPENDENCY Boost
    )
else()
    set(BOOST_DEPENDENCIES
        # Lockfree dependencies
        lockfree core config assert static_assert type_traits mpl preprocessor
        array throw_exception functional align parameter atomic detail
        utility predef winapi
    )

    foreach (dep ${BOOST_DEPENDENCIES})
        RequireExternal(
            TARGET ReverseTale-Library
            EXCLUDE
            SKIP_BUILD
            SKIP_LINK
            MODULE "boostorg/${dep}:develop"
            INC_PATH "include"
        )
    endforeach()
endif()

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

BuildNow(
	TARGET ReverseTale-Library
	DEFINES ${DEFINITIONS}
	STATIC_LIB
	OUTPUT_NAME Library
)


# ---[ If windows, add ws2_32 library
if (MSVC)
	# Somehow it won't work when added as a submodule :/
	if (NOT ${ReverseTale_DEV})
		find_library(SOCKET_LIBRARY ws2_32)
	else()
		set(SOCKET_LIBRARY ws2_32)
	endif()

    target_link_libraries(ReverseTale-Library PUBLIC ${SOCKET_LIBRARY})
endif()

# ---[ Set C++11 standard
set_property(TARGET ReverseTale-Library PROPERTY CXX_STANDARD 11)
target_compile_features(ReverseTale-Library PRIVATE cxx_decltype cxx_nullptr cxx_variadic_templates)

# ---[ Install
include(CMakePackageConfigHelpers)
set(config_install_dir lib/cmake/reversetale-library)
set(version_config ${PROJECT_BINARY_DIR}/reversetale-library-config-version.cmake)
set(project_config ${PROJECT_BINARY_DIR}/reversetale-library-config.cmake)
set(targets_export_name reversetale-library-targets)


# Generate the version, config and target files into the build directory.
write_basic_package_version_file(
  ${version_config}
  VERSION 0.0.1
  COMPATIBILITY AnyNewerVersion)
configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/config.cmake.in
  ${project_config}
  INSTALL_DESTINATION ${config_install_dir})
export(TARGETS ReverseTale-Library FILE ${PROJECT_BINARY_DIR}/${targets_export_name}.cmake)

install(TARGETS ReverseTale-Library
	EXPORT ${targets_export_name}
	RUNTIME DESTINATION bin/
  	LIBRARY DESTINATION lib/
  	ARCHIVE DESTINATION lib/)

install(FILES ${project_config} ${version_config} DESTINATION ${config_install_dir})
install(EXPORT ${targets_export_name} DESTINATION ${config_install_dir})
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include DESTINATION ./)
