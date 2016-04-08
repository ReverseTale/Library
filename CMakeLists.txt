# ---[ Create project
project(ReverseTale-Library)

# ---[ Parse all prefixes
include(cmake-common/Utils.cmake)
String2List(${CMAKE_PREFIX_PATH} CMAKE_PREFIX_PATH)

# ---[ Find all source and header files
file(GLOB_RECURSE sources_Crypto    src/Cryptography/*.cpp     include/Cryptography/*.h)
file(GLOB_RECURSE sources_Tools     src/Tools/*.cpp            include/Tools/*.h)
file(GLOB_RECURSE sources_Game      src/Game/*.cpp             include/Game/*.h)

set(Library_SOURCES
	${sources_Crypto}
	${sources_Tools}
	${sources_Game}
)

# ---[ Load dependencies
find_package(Boost REQUIRED)
find_package(cppformat REQUIRED)

# ---[ Setup include directories
include_directories(
    ${PROJECT_BINARY_DIR}
	${PROJECT_SOURCE_DIR}/include
    ${Boost_INCLUDE_DIR}
)

# ---[ Setup some definitions
set(DEFINITIONS "")
if (NOT ${DEBUG})
  set(DEFINITIONS "${DEFINITIONS} NDEBUG=1")
endif()

# ---[ Add library as static and add dependencies
add_library(ReverseTale-Library STATIC ${Library_SOURCES})

# ---[ Link all third party libraries
target_link_libraries(ReverseTale-Library
    PUBLIC ${Boost_LIBRARY}
    PUBLIC cppformat
)

# ---[ If windows, add ws2_32 library
if (MSVC)
    target_link_libraries(ReverseTale-Library PUBLIC ws2_32)
endif()

# ---[ Set C++11 standard
set_property(TARGET ReverseTale-Library PROPERTY CXX_STANDARD 11)
target_compile_features(ReverseTale-Library PRIVATE cxx_decltype cxx_nullptr cxx_variadic_templates)

# ---[ Setup output directory and include directories
separate_arguments(THIRD_PARTY_INCLUDE)
set_target_properties(ReverseTale-Library PROPERTIES
    OUTPUT_NAME Library
    ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib
    ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/lib
    ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${PROJECT_SOURCE_DIR}/lib

    INTERFACE_COMPILE_DEFINITIONS "${DEFINITIONS}"
    INTERFACE_INCLUDE_DIRECTORIES "$<INSTALL_INTERFACE:include>;${Boost_INCLUDE_DIR}"
)

include(CMakePackageConfigHelpers)
set(config_install_dir lib/cmake/ReverseTale-Library)
set(version_config ${PROJECT_BINARY_DIR}/ReverseTale-Library-config-version.cmake)
set(project_config ${PROJECT_BINARY_DIR}/ReverseTale-Library-config.cmake)
set(targets_export_name ReverseTale-Library-targets)


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
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include DESTINATION include/)
