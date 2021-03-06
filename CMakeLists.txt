cmake_minimum_required (VERSION 3.5)
project (Dataframe
         VERSION 1.0.0)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # Produce compile_commands.json
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# Library type
if(MSVC)
option(STATIC "Set to ON to build Dataframe as a static library instead of a shared library" ON)
add_definitions(-D_CRT_SECURE_NO_WARNINGS)
else(MSVC)
option(STATIC "Set to ON to build Dataframe as a static library instead of a shared library" OFF)
	if(NOT UNIX)
		add_compile_options(-Wa,-mbig-obj)
	endif(NOT UNIX)
endif(MSVC)

if(STATIC)
	if(NOT MSVC)
	add_compile_options(-static)
	endif(NOT MSVC)
else(STATIC)
	if(MSVC)
		# # To build shared libraries in Windows, we set CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS to TRUE.
		# # See https://cmake.org/cmake/help/v3.4/variable/CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS.html
		# # See https://blog.kitware.com/create-dlls-on-windows-without-declspec-using-new-cmake-export-all-feature/
		set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
	endif(MSVC)
endif(STATIC)

# We use
# - InstallBasicPackageFiles (http://robotology.github.io/ycm/gh-pages/v0.8/module/InstallBasicPackageFiles.html)
# - AddUninstallTarget (http://robotology.github.io/ycm/gh-pages/v0.8/module/AddUninstallTarget.html)
# - AddInstallRPATHSupport (http://robotology.github.io/ycm/gh-pages/v0.8/module/AddInstallRPATHSupport.html)
# See https://github.com/robotology/ycm/
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

# Control where libraries and executables are placed during the build.
# With the following settings executables are placed in <the top level of the
# build tree>/bin and libraries/archives in <top level of the build tree>/lib.
# This is particularly useful to run ctests on libraries built on Windows
# machines: tests, which are executables, are placed in the same folders of
# dlls, which are treated as executables as well, so that they can properly
# find the libraries to run. This is a because of missing RPATH on Windows.
include(GNUInstallDirs)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_BINDIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR}")

# Disable C and C++ compiler extensions.
# C/CXX_EXTENSIONS are ON by default to allow the compilers to use extended
# variants of the C/CXX language.
# However, this could expose cross-platform bugs in user code or in the headers
# of third-party dependencies and thus it is strongly suggested to turn
# extensions off.
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_CXX_EXTENSIONS OFF)

# Enable RPATH support for installed binaries and libraries
include(AddInstallRPATHSupport)
add_install_rpath_support(BIN_DIRS "${CMAKE_INSTALL_FULL_LIBDIR}"
                          LIB_DIRS "${CMAKE_INSTALL_FULL_BINDIR}"
                          INSTALL_NAME_DIR "${CMAKE_INSTALL_FULL_LIBDIR}"
                          USE_LINK_PATH)

# Encourage user to specify a build type (e.g. Release, Debug, etc.), otherwise set it to Release.
if(NOT CMAKE_CONFIGURATION_TYPES)
    if(NOT CMAKE_BUILD_TYPE)
        message(STATUS "Setting build type to 'Release' as none was specified.")
        set_property(CACHE CMAKE_BUILD_TYPE PROPERTY VALUE "Release")
    endif()
endif()

set(LIBRARY_TARGET_NAME ${PROJECT_NAME})

add_subdirectory(DataFrame)
add_subdirectory(test)

# Enable ctest, testing so we can see if unit tests pass or fail in CI
enable_testing()
add_test(NAME dataframe_tester
		COMMAND dataframe_tester
		WORKING_DIRECTORY $<TARGET_FILE_DIR:dataframe_tester>)
add_test(NAME vectors_tester
		COMMAND vectors_tester
		WORKING_DIRECTORY $<TARGET_FILE_DIR:vectors_tester>)
add_test(NAME date_time_tester
		COMMAND date_time_tester
		WORKING_DIRECTORY $<TARGET_FILE_DIR:date_time_tester>)

message("-- Copying files for testing")
# Ctest require this files in the build dir, on all platforms
file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data_dt_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data_string_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
if (MSVC)
	file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Debug)
	file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Release)
	file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data_dt_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Debug)
	file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data_dt_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Release)
	file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data_string_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Debug)
	file(COPY ${PROJECT_SOURCE_DIR}/test/sample_data_string_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Release)
endif(MSVC)
message("-- Copying files for testing - done")
