# =======================================================================
#   WARNING    WARNING    WARNING    WARNING    WARNING    WARNING
# =======================================================================
#   Remember to put on SAFETY GOGGLES before looking at this file. You
#   are most certainly not expected to read or understand any of it.
# =======================================================================
# This CMake file is responsible for compiling dependency libraries and
# setting up suitable compiler flags for various platforms. You do not
# need to read or change anything in this file; see CMakeLists.txt 


# Top-level External Libraries 3 CMake file, requires a recent version.
cmake_minimum_required (VERSION 3.23...3.31)

project(external
  DESCRIPTION
    "External Libraries"
  LANGUAGES
    CXX C
)

#Check to make sure the project was cloned correctly with submodules
if(NOT IS_DIRECTORY "${CMAKE_SOURCE_DIR}/external_libraries/eigen")
  message(FATAL_ERROR "The Eigen submodule directory is missing! "
    "You probably did not clone the project with --recursive, or first checked out "
    "External Libraries before it was added. It is possible to recover by running "
    "\"git submodule update --init --recursive\"")
endif()



# Add paths to our CMake code to the module path, so they can be found automatically by
# CMake.
set(CMAKE_MODULE_PATH
  ${CMAKE_MODULE_PATH}
  )

# Determine information about the compiler
# Sets some variables depending on which compiler you are using
# USING_GNU_C       : gcc is being used for C compiler
# USING_GNU_CXX     : g++ is being used for C++ compiler
# USING_CLANG_C     : gcc is being used for C compiler
# USING_CLANG_CXX   : g++ is being used for C++ compiler
# USING_ICC         : icc is being used for C compiler
# USING_ICPC        : icpc is being used for C++ compiler
# USING_WINDOWS_CL  : Visual Studio's compiler
# USING_WINDOWS_ICL : Intel's Windows compiler

set(USING_KNOWN_C_COMPILER TRUE)
if(CMAKE_COMPILER_IS_GNUCC)
  set(USING_GNU_C TRUE)
elseif( CMAKE_C_COMPILER_ID STREQUAL "Intel" )
  set(USING_ICC TRUE)
elseif( CMAKE_C_COMPILER_ID MATCHES "Clang" )
  set(USING_CLANG_C TRUE)
elseif( MSVC OR "x${CMAKE_C_COMPILER_ID}" STREQUAL "xMSVC" )
  set(USING_WINDOWS_CL TRUE)
else()
  set(USING_KNOWN_C_COMPILER FALSE)
endif()


set(USING_KNOWN_CXX_COMPILER TRUE)
if(CMAKE_COMPILER_IS_GNUCXX)
  set(USING_GNU_CXX TRUE)
elseif( CMAKE_CXX_COMPILER_ID STREQUAL "Intel" )
  set(USING_ICPC TRUE)
elseif( CMAKE_CXX_COMPILER_ID MATCHES "Clang" )
  set(USING_CLANG_CXX TRUE)
elseif( MSVC OR "x${CMAKE_C_COMPILER_ID}" STREQUAL "xMSVC" )
  if( NOT USING_WINDOWS_CL )
    message( WARNING "Mixing WinCL C++ compiler with non-matching C compiler" )
  endif()
else()
  set(USING_KNOWN_CXX_COMPILER FALSE)
endif()

if(USING_GNU_C)
  execute_process(COMMAND ${CMAKE_C_COMPILER} -dumpversion
    OUTPUT_VARIABLE GCC_VERSION)
endif()

# Using unknown compilers
if(NOT USING_KNOWN_C_COMPILER)
  FIRST_TIME_MESSAGE("Specified C compiler ${CMAKE_C_COMPILER} is not recognized (gcc, icc).  Using CMake defaults.")
endif()

if(NOT USING_KNOWN_CXX_COMPILER)
  FIRST_TIME_MESSAGE("Specified CXX compiler ${CMAKE_CXX_COMPILER} is not recognized (g++, icpc).  Using CMake defaults.")
endif()


if(USING_WINDOWS_CL)
  # We should set this macro as well to get our nice trig functions
  add_definitions(-D_USE_MATH_DEFINES)
  # Microsoft does some stupid things like #define min and max.
  add_definitions(-DNOMINMAX)
endif()
  

# Determine if the current compiler supports the C++11 'override' keyword
if( MSVC10 OR MSVC12 ) # TODO: make this more robust
  set( SUPPORTS_OVERRIDE TRUE )
else()
  set( SUPPORTS_OVERRIDE FALSE )
endif()


# Check for specific machine/compiler options.
include(CheckCXXCompilerFlag)
include(CheckCXXSourceRuns)


# Put all the runtime stuff in the same directory.  By default, CMake puts each targets'
# output into their own directory.  We want all the targets to be put in the same
# directory, and we can do this by setting these variables.
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}" CACHE PATH "Directory where .exe and library files are dumped." FORCE)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}" CACHE PATH "Directory where .exe and library files are dumped." FORCE)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}" CACHE PATH "Directory where .exe and library files are dumped." FORCE)


# Set the default build to Release.  Note this doesn't do anything for the VS
# default build target which defaults to Debug when you first start it.
# Set a default build configuration (Release)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to 'Release' as none was specified.")
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release"
    "MinSizeRel" "RelWithDebInfo")
endif()
string(TOUPPER "${CMAKE_BUILD_TYPE}" U_CMAKE_BUILD_TYPE)


# Enable folders for projects in Visual Studio
if (CMAKE_GENERATOR MATCHES "Visual Studio")
  set_property(GLOBAL PROPERTY USE_FOLDERS ON )
endif()

if (APPLE)
  set(CMAKE_MACOSX_RPATH ON PARENT_SCOPE)
endif()

set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)


# Sanitize build environment for static build with C++11
if (MSVC)
  # Disable annoying secure CRT warnings
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /D_CRT_SECURE_NO_WARNINGS")

  # We'll select the TBB library ourselves
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /D__TBB_NO_IMPLICIT_LINKAGE")

  # Parallel build on MSVC (all targets)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")

  if (NOT CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE2")

    # Disable Eigen vectorization for Windows 32 bit builds (issues with unaligned access segfaults)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /DEIGEN_DONT_ALIGN")
  endif()

  # Static build
  set(CompilerFlags
        CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE
        CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO
        CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE
        CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO)
  foreach(CompilerFlag ${CompilerFlags})
    string(REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")
  endforeach()
elseif(APPLE)
  # Try to auto-detect a suitable SDK
  execute_process(COMMAND bash -c "xcodebuild -version -sdk | grep MacOSX | grep Path | head -n 1 | cut -f 2 -d ' '" OUTPUT_VARIABLE CMAKE_OSX_SYSROOT)
  string(REGEX REPLACE "(\r?\n)+$" "" CMAKE_OSX_SYSROOT "${CMAKE_OSX_SYSROOT}")
  string(REGEX REPLACE "^.*X([0-9.]*).sdk$" "\\1" CMAKE_OSX_DEPLOYMENT_TARGET "${CMAKE_OSX_SYSROOT}")
endif()

# check for C++11 Support
if (CMAKE_CXX_COMPILER_ID MATCHES "^(GNU|Clang|Intel)$")
  CHECK_CXX_COMPILER_FLAG("-std=c++14" HAS_CPP14_FLAG)
  CHECK_CXX_COMPILER_FLAG("-std=c++11" HAS_CPP11_FLAG)

  if (HAS_CPP14_FLAG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
  elseif (HAS_CPP11_FLAG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
  else()
    message(FATAL_ERROR "Unsupported compiler -- This project requires C++11 support!")
  endif()

  # Enable link time optimization and set the default symbol
  # visibility to hidden (very important to obtain small binaries)
  if (NOT ${U_CMAKE_BUILD_TYPE} MATCHES DEBUG)
    # Default symbol visibility
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fvisibility=hidden")
  endif()

endif()

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")


# Compile remainder of the codebase with compiler warnings turned on
if(MSVC)
  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  endif()
elseif (CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wno-unused-parameter")
  if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-gnu-anonymous-struct -Wno-c99-extensions -Wno-nested-anon-types -Wno-deprecated-register")
  endif()
endif()


SET(REBUILD_ALL FALSE)

ADD_SUBDIRECTORY(external_libraries)