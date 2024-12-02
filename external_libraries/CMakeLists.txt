# =======================================================================
#   WARNING    WARNING    WARNING    WARNING    WARNING    WARNING
# =======================================================================
#   Remember to put on SAFETY GOGGLES before looking at this file. You
#   are most certainly not expected to read or understand any of it.
# =======================================================================
# This CMake file is responsible for compiling dependency libraries and
# setting up suitable compiler flags for various platforms. You do not
# need to read or change anything in this file; see CMakeLists.txt 

#Build Zlib ##################################################################
IF(WIN32) 
   SET(CMAKE_DEBUG_POSTFIX "d" CACHE STRING "Add a suffix, usually d on Windows" FORCE)
   set(ZLIB_ROOT ${CMAKE_BINARY_DIR}/Debug ${CMAKE_BINARY_DIR}/Release  ${CMAKE_CURRENT_SOURCE_DIR}/zlib)
else()
   set(ZLIB_ROOT ${CMAKE_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/zlib)
endif()

set(ZLIB_BUILD_EXAMPLES OFF CACHE BOOL "Disable ZLIB example builds")
include_directories(${CMAKE_SOURCE_DIR}/external_libraries/zlib)

ADD_SUBDIRECTORY(zlib)
if(NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/zlib/zconf.h)
  message(STATUS "Moving zconf back since who knows why its moved in the first place...")
  file(RENAME ${CMAKE_CURRENT_SOURCE_DIR}/zlib/zconf.h.included ${CMAKE_CURRENT_SOURCE_DIR}/zlib/zconf.h)
endif()

IF(WIN32) 
  SET(ZLIB_LIBRARY_RELEASE "${CMAKE_BINARY_DIR}/$(CONFIGURATION)/zlib.lib")
  SET(ZLIB_LIBRARY_DEBUG "${CMAKE_BINARY_DIR}/$(CONFIGURATION)/zlibd.lib")
ENDIF()
find_package(ZLIB)
if(NOT ${ZLIB_FOUND})
  message(STATUS "Setting ZLIB variables manually...")
  IF(WIN32) 
    SET(ZLIB_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/zlib")
    include(${CMAKE_SOURCE_DIR}/CMake/SelectLibraryConfigurations.cmake)
    select_library_configurations(ZLIB)
    add_definitions(-DWIN32)
  ELSE()
     SET(ZLIB_LIBRARY z CACHE INTERNAL "" FORCE)
  ENDIF()
  SET(ZLIB_INCLUDE "${CMAKE_CURRENT_SOURCE_DIR}/zlib" CACHE INTERNAL "" FORCE)
ENDIF()

Message(STATUS " ZLIB has been found... " ${ZLIB_FOUND})
Message(STATUS " ZLIB version: " ${ZLIB_VERSION_STRING})
Message(STATUS " -ZLIB_INCLUDE_DIRS is set to: " ${ZLIB_INCLUDE_DIRS})
Message(STATUS " -ZLIB_LIBRARY is set to: " ${ZLIB_LIBRARY})
Message(STATUS " -ZLIB_LIBRARY-rr is set to: " ${ZLIB_LIBRARY_RELEASE})
Message(STATUS " -ZLIB_LIBRARY--dd is set to: " ${ZLIB_LIBRARY_DEBUG})
message(STATUS "ZLIB ROOT: " ${ZLIB_ROOT})



# ILMBase ##############################################################
OPTION (NAMESPACE_VERSIONING "Namespace Versioning" OFF)
ADD_SUBDIRECTORY(Imath)



###Build openexr ##################################################################
if (CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
  add_compile_options(-Wno-unused-parameter -Wno-unused-but-set-variable -Wno-sign-compare)
endif()

# Prevents openexr to set the DEBUG_POSTFIX "_d"
set(CMAKE_DEBUG_POSTFIX "" CACHE STRING " " FORCE)
set(ILMBASE_LIB_SUFFIX  "" CACHE STRING "" FORCE)
set(OPENEXR_LIB_SUFFIX  "" CACHE STRING "" FORCE)
set(OPENEXR_BUILD_UTILS        OFF CACHE BOOL "" FORCE)
set(ILMBASE_INSTALL_PKG_CONFIG OFF CACHE BOOL "" FORCE)
set(OPENEXR_INSTALL_PKG_CONFIG OFF CACHE BOOL "" FORCE)
set(OPENEXR_TEST_LIBRARIES           OFF CACHE BOOL "" FORCE)
set(OPENEXR_BUILD_EXAMPLES          OFF CACHE BOOL "" FORCE)
set(OPENEXR_TEST_PYTHON           OFF CACHE BOOL "" FORCE)
set(PYILMBASE_ENABLE           OFF CACHE BOOL "" FORCE)
set(PYILMBASE_ENABLE           OFF CACHE BOOL "" FORCE)
set(INSTALL_OPENEXR_DOCS OFF CACHE BOOL "" FORCE)
set(INSTALL_OPENEXR_EXAMPLES OFF CACHE BOOL "" FORCE)
set(BUILD_TESTING OFF CACHE BOOL "" FORCE)

set(CMAKE_CXX_STANDARD 17)
unset(CMAKE_CXX_VISIBILITY_PRESET)
add_subdirectory(openexr)
set(CMAKE_CXX_VISIBILITY_PRESET "hidden")




#if (MSVC)
#  target_compile_options(Half PRIVATE /W2)
#  target_compile_options(IlmThread PRIVATE /W2)
#  target_compile_options(Imath PRIVATE /W2)
#  target_compile_options(IlmImf PRIVATE /W1)
#endif()

#set(OPENEXR_INCLUDE_DIRS
#  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Imath
#  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Imath
#  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Imath
#  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Iex
#  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/IlmBase/Half
#  ${CMAKE_CURRENT_SOURCE_DIR}/openexr/OpenEXR/IlmImf
#  ${CMAKE_CURRENT_BINARY_DIR}/openexr/OpenEXR/config
#  ${CMAKE_CURRENT_BINARY_DIR}/openexr/IlmBase/config
#  PARENT_SCOPE)

mark_as_advanced(
  ILMBASE_BUILD_BOTH_STATIC_SHARED ILMBASE_ENABLE_LARGE_STACK
  ILMBASE_IEX_NAMESPACE ILMBASE_ILMTHREAD_NAMESPACE ILMBASE_IMATH_NAMESPACE
  ILMBASE_INSTALL_PKG_CONFIG ILMBASE_INTERNAL_IEX_NAMESPACE
  ILMBASE_INTERNAL_ILMTHREAD_NAMESPACE ILMBASE_INTERNAL_IMATH_NAMESPACE
  ILMBASE_LIB_SUFFIX ILMBASE_NAMESPACE_CUSTOM ILMBASE_OUTPUT_SUBDIR
  ILMBASE_PACKAGE_NAME ILMBASE_STATIC_LIB_SUFFIX IlmBase_DIR PYILMBASE_ENABLE
)

mark_as_advanced(
  OPENEXR_BUILD_BOTH_STATIC_SHARED OPENEXR_BUILD_UTILS OPENEXR_CXX_STANDARD
  OPENEXR_IMF_NAMESPACE OPENEXR_INSTALL_PKG_CONFIG OPENEXR_INTERNAL_IMF_NAMESPACE
  OPENEXR_LIB_SUFFIX OPENEXR_NAMESPACE_CUSTOM OPENEXR_OUTPUT_SUBDIR
  OPENEXR_PACKAGE_NAME OPENEXR_RUN_FUZZ_TESTS OPENEXR_STATIC_LIB_SUFFIX
  OPENEXR_USE_CLANG_TIDY OpenEXR_DIR INSTALL_OPENEXR_DOCS
  INSTALL_OPENEXR_EXAMPLES
)





###Build libpng ##################################################################
#set(PNG_SHARED ON CACHE BOOL " " FORCE)
#set(PNG_STATIC OFF CACHE BOOL " " FORCE)
#set(PNG_TESTS OFF CACHE BOOL " " FORCE)
#set(PNG_SKIP_INSTALL_ALL TRUE)
#if (CMAKE_SYSTEM_PROCESSOR STREQUAL "arm64")
#set(PNG_ARM_NEON "on" CACHE STRING " " FORCE)
#endif()
#add_subdirectory(libpng)
#
#set(PNG_INCLUDE_DIRS "${CMAKE_CURRENT_SOURCE_DIR}/libpng;${CMAKE_CURRENT_BINARY_DIR}/libpng" PARENT_SCOPE)
#set(PNG_LIBRARIES    "png" PARENT_SCOPE)
#set(PNG_DEFINES      -DMI_HAS_LIBPNG PARENT_SCOPE)
#
#mark_as_advanced(DFA_XTRA AWK PNG_FRAMEWORK PNG_HARDWARE_OPTIMIZATIONS
#  PNG_PREFIX PNG_SHARED PNG_STATIC PNG_TESTS PNG_BUILD_ZLIB PNG_DEBUG
#  PNG_INTEL_SSE M_LIBRARY ld-version-script)
#
#add_dependencies(png-fix-itxt zlib)
#add_dependencies(pngfix zlib)
#add_dependencies(png_genfiles zlib)
#
#
#
##Build cnpy ##################################################################
#set(CNPY_LIBRARIES  cnpy)
#IF(WIN32) 
#    set(ZLIB_LIBRARIES  zlib)
#  ELSE()
#    set(ZLIB_LIBRARIES  z)
#ENDIF()
#set(ZLIB_INCLUDE_DIRS  ${CMAKE_SOURCE_DIR}/external_libraries/zlib)
#set(CNPY_INCLUDE_DIR  ${CMAKE_SOURCE_DIR}/external_libraries/cnpy)
#SET(NUMPY_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/cnpy)
#add_library(cnpy STATIC "${NUMPY_INCLUDE_DIR}/cnpy.cpp" "${NUMPY_INCLUDE_DIR}/cnpy.h")
#target_include_directories(cnpy PRIVATE ${NUMPY_INCLUDE_DIR} ${ZLIB_INCLUDE_DIRS})
#add_library(cnpy-shared SHARED "${NUMPY_INCLUDE_DIR}/cnpy.cpp" "${NUMPY_INCLUDE_DIR}/cnpy.h")
#target_include_directories(cnpy-shared PRIVATE ${NUMPY_INCLUDE_DIR} ${ZLIB_INCLUDE_DIRS})
#target_link_libraries(cnpy-shared ${ZLIB_LIBRARIES})
#
#
##Build xz ##################################################################
#set(BUILD_TESTING OFF CACHE BOOL "Disable XZ example builds")
#ADD_SUBDIRECTORY(xz)
#
#
##Build glm ##################################################################
#ADD_SUBDIRECTORY(glm)
#
#
##Include GLOG  ################################################################
#option(WITH_GFLAGS "Use GFLAGS" OFF )
#option(USE_GFLAGS "Use GFLAGS" OFF)
#set(BUILD_EXAMPLES OFF CACHE BOOL "Disable glog example builds")
#if(USE_GFLAGS)
#  include("cmake/External/gflags.cmake")
#  if(GFLAGS_FOUND)
#    add_definitions(-DCAFFE2_USE_GFLAGS)
#    include_directories(SYSTEM ${GFLAGS_INCLUDE_DIRS})
#    list(APPEND Caffe2_DEPENDENCY_LIBS ${GFLAGS_LIBRARIES})
#  else()
#    message(WARNING "Not compiling with gflags. Suppress this warning with -DUSE_GFLAGS=OFF")
#    set(USE_GFLAGS OFF)
#  endif()
#endif()
#SET(WITH_GFLAGS OFF CACHE BOOL "Use gflags" FORCE)
#add_subdirectory(glog)
#SET_PROPERTY(TARGET utilities_unittest stl_logging_unittest glog logging_unittest demangle_unittest PROPERTY FOLDER "GLog")
#
#
##Build glm ##################################################################
#ADD_SUBDIRECTORY(googletest)
#
#
#Build fast_float ##################################################################
#ADD_SUBDIRECTORY(fast_float)

