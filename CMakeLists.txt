cmake_minimum_required (VERSION 3.5)
project (Dataframe
         VERSION 1.0.0)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON) # Produce compile_commands.json
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

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

# # To build shared libraries in Windows, we set CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS to TRUE.
# # See https://cmake.org/cmake/help/v3.4/variable/CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS.html
# # See https://blog.kitware.com/create-dlls-on-windows-without-declspec-using-new-cmake-export-all-feature/
# set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

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

# List of source files.
set(${LIBRARY_TARGET_NAME}_SRC
  src/HeteroVector.cc
  src/HeteroView.cc
  src/ThreadGranularity.cc
  src/DateTime.cc
)

# List of header files.
set(${LIBRARY_TARGET_NAME}_HDR
  include/VectorView.h
  include/HeteroVector.h
  include/HeteroView.h
  include/HeteroVector.tcc
  include/HeteroView.tcc
  include/DataFrameVisitors.h
  include/DataFrameOperators.h
  include/DataFrame.h
  include/DataFrame.tcc
  include/DataFrame_get.tcc
  include/DataFrame_misc.tcc
  include/DataFrame_opt.tcc
  include/DataFrame_join.tcc
  include/DataFrame_shift.tcc
  include/DataFrame_read.tcc
  include/DataFrame_set.tcc
  include/DataFrame_functors.h
  include/ThreadGranularity.h
  include/DateTime.h
  include/FixedSizeString.h
)

# Build the library
add_definitions(-D DMS_INCLUDE_SOURCE) # Need this for some of the .tcc files to be compiled
if (UNIX)
  add_library(${LIBRARY_TARGET_NAME} SHARED ${${LIBRARY_TARGET_NAME}_SRC})
endif(UNIX)

if (MSVC)
  add_library(${LIBRARY_TARGET_NAME} STATIC ${${LIBRARY_TARGET_NAME}_SRC})
endif(MSVC)


# Set two minimum target properties for the library.
# See https://cmake.org/cmake/help/latest/command/set_target_properties.html
# Properties are: 1) Library version number 2) list of corresponding public headers
set_target_properties(${LIBRARY_TARGET_NAME} PROPERTIES VERSION       ${${PROJECT_NAME}_VERSION}
                                                        PUBLIC_HEADER "${${LIBRARY_TARGET_NAME}_HDR}")

# Specify include directories for both compilation and installation process.
# The $<INSTALL_PREFIX> generator expression is useful to ensure to create
# relocatable configuration files, see https://cmake.org/cmake/help/latest/manual/cmake-packages.7.html#creating-relocatable-packages
target_include_directories(${LIBRARY_TARGET_NAME} PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>"
							 "$<INSTALL_INTERFACE:$<INSTALL_PREFIX>/${CMAKE_INSTALL_INCLUDEDIR}>")

# Specify installation targets, typology and destination folders.
install(TARGETS ${LIBRARY_TARGET_NAME}
        EXPORT  ${PROJECT_NAME}
        LIBRARY       DESTINATION "${CMAKE_INSTALL_LIBDIR}"                            COMPONENT shlib
        ARCHIVE       DESTINATION "${CMAKE_INSTALL_LIBDIR}"                            COMPONENT lib
        RUNTIME       DESTINATION "${CMAKE_INSTALL_BINDIR}"                            COMPONENT bin
        PUBLIC_HEADER DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${LIBRARY_TARGET_NAME}" COMPONENT dev)

# Create and install CMake configuration files for your project that are
# necessary to for other projects to call find_package().
#
# Note that it is extremely important to use exactly the project name while installing configuration
# files (you can use PROJECT_NAME variable to avoid any possible error). This is required to allow
# find_package() to properly look for the installed library in system path, in particular in Windows
# when the installation is performed in the default path.
#
# install_basic_package_files() comes with many input parameters to customize the configuration
# files. The parameters used in the following call provide basic versions of CMake configuration
# files.  See install_basic_package_files() documentation found in ./cmake folder.
#
# Note that if your library depends from other libraries, you are probably
# required to used the install_basic_package_files() DEPENDENCIES option.
include(InstallBasicPackageFiles)
if (UNIX)
  install_basic_package_files(${PROJECT_NAME}
                            VERSION ${${PROJECT_NAME}_VERSION}
                            COMPATIBILITY AnyNewerVersion
                            EXPORT ${PROJECT_NAME}
                            VARS_PREFIX ${PROJECT_NAME}
                            NO_CHECK_REQUIRED_COMPONENTS_MACRO)
endif (UNIX)
if (MSVC)
  install_basic_package_files(${PROJECT_NAME}
                            VERSION ${${PROJECT_NAME}_VERSION}
                            COMPATIBILITY AnyNewerVersion
                            EXPORT ${PROJECT_NAME}
                            VARS_PREFIX ${PROJECT_NAME}
                            NO_CHECK_REQUIRED_COMPONENTS_MACRO)
endif(MSVC)

# Add the uninstall target
include(AddUninstallTarget)

# Build the test binary
add_executable(dataframe_tester src/dataframe_tester.cc)
add_executable(vectors_tester src/vectors_tester.cc)
add_executable(date_time_tester src/date_time_tester.cc)
target_link_libraries(dataframe_tester Dataframe) # Link the Dataframe library to the test binary
target_link_libraries(vectors_tester Dataframe) # Link the Dataframe library to the test binary
target_link_libraries(date_time_tester Dataframe) # Link the Dataframe library to the test binary
if (UNIX)
  # Find pthreads library
  set(THREADS_PREFER_PTHREAD_FLAG ON)
  find_package(Threads REQUIRED)
  target_link_libraries(dataframe_tester Threads::Threads)
  target_link_libraries(vectors_tester Threads::Threads)
  target_link_libraries(date_time_tester Threads::Threads)
endif (UNIX)

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
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data_dt_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data_string_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
if (MSVC)
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Debug)
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Release)
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data_dt_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Debug)
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data_dt_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Release)
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data_string_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Debug)
  file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/test/sample_data_string_index.csv DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/bin/Release)
endif(MSVC)
message("-- Copying files for testing - done")
