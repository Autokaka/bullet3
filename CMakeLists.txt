cmake_minimum_required(VERSION 3.8.0)
set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)
set(MSVC_INCREMENTAL_DEFAULT ON)
project(bullet3)

# Exposed options
option(USE_DOUBLE_PRECISION "Use double precision" OFF)
option(USE_GRAPHICAL_BENCHMARK "Use Graphical Benchmark" OFF)
option(BUILD_SHARED_LIBS "Use shared libraries" OFF)
option(USE_SOFT_BODY_MULTI_BODY_DYNAMICS_WORLD "Use btSoftMultiBodyDynamicsWorld" ON)
option(BULLET2_MULTITHREADING "Build Bullet 2 libraries with mutex locking around certain operations (required for multi-threading)" OFF)
if(BULLET2_MULTITHREADING)
  option(BULLET2_USE_OPEN_MP_MULTITHREADING "Build Bullet 2 with support for multi-threading with OpenMP (requires a compiler with OpenMP support)" OFF)
  option(BULLET2_USE_TBB_MULTITHREADING "Build Bullet 2 with support for multi-threading with Intel Threading Building Blocks (requires the TBB library to be already installed)" OFF)
  if(MSVC)
    option(BULLET2_USE_PPL_MULTITHREADING "Build Bullet 2 with support for multi-threading with Microsoft Parallel Patterns Library (requires MSVC compiler)" OFF)
  endif()
endif()
option(USE_MSVC_INCREMENTAL_LINKING "Use MSVC Incremental Linking" OFF)
if(CMAKE_SYSTEM_NAME STREQUAL WindowsPhone OR CMAKE_SYSTEM_NAME STREQUAL WindowsStore)
	option(USE_MSVC_RUNTIME_LIBRARY_DLL "Use MSVC Runtime Library DLL (/MD or /MDd)" ON)
else()
	option(USE_MSVC_RUNTIME_LIBRARY_DLL "Use MSVC Runtime Library DLL (/MD or /MDd)" OFF)
endif()
option(USE_MSVC_RELEASE_RUNTIME_ALWAYS "Use MSVC Release Runtime Library even in Debug" OFF)

# Global settings
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -D_DEBUG")
set(CLAMP_VELOCITIES "0" CACHE string "Clamp rigid bodies' velocity to this value, if larger than zero. Useful to prevent floating point errors or in general runaway velocities in complex scenarios")
if(MSVC)
  if(NOT USE_MSVC_INCREMENTAL_LINKING)
    set(MSVC_INCREMENTAL_YES_FLAG "/INCREMENTAL:NO")
    string(REPLACE "INCREMENTAL:YES" "INCREMENTAL:NO" REPLACE_FLAGS "${CMAKE_EXE_LINKER_FLAGS_DEBUG}")
    set(CMAKE_EXE_LINKER_FLAGS_DEBUG "/INCREMENTAL:NO ${REPLACE_FLAGS}")
    string(REPLACE "INCREMENTAL:YES" "INCREMENTAL:NO" REPLACE_FLAGS2 "${CMAKE_EXE_LINKER_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS ${REPLACE_FLAGS2})
    string(REPLACE "INCREMENTAL:YES" "" REPLACE_FLAGS3 "${CMAKE_EXTRA_LINK_FLAGS}")
    string(CMAKE_EXTRA_LINK_FLAGS ${REPLACE_FLAGS3})
    string(REPLACE "INCREMENTAL:YES" "INCREMENTAL:NO" REPLACE_FLAGS3 "${CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO}")
    set(CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO ${REPLACE_FLAGS3})
    set(CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO "/INCREMENTAL:NO ${REPLACE_FLAGS3}" )
  endif()
  if (NOT USE_MSVC_RUNTIME_LIBRARY_DLL)
    foreach(FLAG_VAR CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO)
    if(${FLAG_VAR} MATCHES "/MD")
      string(REGEX REPLACE "/MD" "/MT" ${FLAG_VAR} "${${FLAG_VAR}}")
    endif()
    if(${FLAG_VAR} MATCHES "/MDd")
      string(REGEX REPLACE "/MDd" "/MTd" ${FLAG_VAR} "${${FLAG_VAR}}")
    endif()
    endforeach()
	endif()
	if (USE_MSVC_RELEASE_RUNTIME_ALWAYS)
    foreach(FLAG_VAR CMAKE_CXX_FLAGS CMAKE_CXX_FLAGS_DEBUG CMAKE_CXX_FLAGS_RELEASE CMAKE_CXX_FLAGS_MINSIZEREL CMAKE_CXX_FLAGS_RELWITHDEBINFO CMAKE_C_FLAGS CMAKE_C_FLAGS_DEBUG CMAKE_C_FLAGS_RELEASE CMAKE_C_FLAGS_MINSIZEREL CMAKE_C_FLAGS_RELWITHDEBINFO )
      if(${FLAG_VAR} MATCHES "/MDd")
        string(REGEX REPLACE "/MDd" "/MD" ${FLAG_VAR} "${${FLAG_VAR}}")
      endif(${FLAG_VAR} MATCHES "/MDd")
      if(${FLAG_VAR} MATCHES "/MTd")
        string(REGEX REPLACE "/MTd" "/MT" ${FLAG_VAR} "${${FLAG_VAR}}")
      endif(${FLAG_VAR} MATCHES "/MTd")
      if(${FLAG_VAR} MATCHES "-D_DEBUG")
        string(REGEX REPLACE "-D_DEBUG" "" ${FLAG_VAR} "${${FLAG_VAR}}")
      endif(${FLAG_VAR} MATCHES "-D_DEBUG")
    endforeach()
	endif(USE_MSVC_RELEASE_RUNTIME_ALWAYS)
  if(CMAKE_SIZEOF_VOID_P EQUAL 64)
    add_definitions(-D_WIN64)
  else()
    option(USE_MSVC_SSE "Use MSVC /arch:sse option"	OFF)
    option(USE_MSVC_SSE2 "Compile your program with SSE2 instructions" ON)
    if(USE_MSVC_SSE)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE")
    endif()
    if(USE_MSVC_SSE2)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE2")
    endif()
  endif()
	option(USE_MSVC_AVX "Compile your program with AVX instructions"  OFF)
	if(USE_MSVC_AVX)
		add_definitions(/arch:AVX)
	endif()
  option(USE_MSVC_FAST_FLOATINGPOINT "Use MSVC /fp:fast option"	ON)
  if(USE_MSVC_FAST_FLOATINGPOINT)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /fp:fast")
  endif()
	option(USE_MSVC_STRING_POOLING "Use MSVC /GF string pooling option"	ON)
	if(USE_MSVC_STRING_POOLING)
		set(CMAKE_C_FLAGS "/GF ${CMAKE_C_FLAGS}")
		set(CMAKE_CXX_FLAGS "/GF ${CMAKE_CXX_FLAGS}")
	endif()
	option(USE_MSVC_FUNCTION_LEVEL_LINKING "Use MSVC /Gy function level linking option"	ON)
	if(USE_MSVC_FUNCTION_LEVEL_LINKING)
		set(CMAKE_C_FLAGS "/Gy ${CMAKE_C_FLAGS}")
		set(CMAKE_CXX_FLAGS "/Gy ${CMAKE_CXX_FLAGS}")
		set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /OPT:REF")
		set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /OPT:REF")
	endif()
	option(USE_MSVC_EXEPTIONS "Use MSVC C++ exceptions option" OFF)
	option(USE_MSVC_COMDAT_FOLDING "Use MSVC /OPT:ICF COMDAT folding option" ON)
	if(USE_MSVC_COMDAT_FOLDING)
		set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} /OPT:ICF")
		set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} /OPT:ICF")
	endif()
	option(USE_MSVC_DISABLE_RTTI "Use MSVC /GR- disabled RTTI flags option"	ON)
  if(USE_MSVC_DISABLE_RTTI)
    string(REGEX REPLACE "/GR" "" CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS}) # Disable RTTI
    set(CMAKE_C_FLAGS "/GR- ${CMAKE_C_FLAGS}")
    set(CMAKE_CXX_FLAGS "/GR- ${CMAKE_CXX_FLAGS}")
  endif()
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /wd4244 /wd4267")
endif()

if(WIN32)
  option(INTERNAL_CREATE_DISTRIBUTABLE_MSVC_PROJECTFILES "Create MSVC projectfiles that can be distributed" OFF)
  if(INTERNAL_CREATE_DISTRIBUTABLE_MSVC_PROJECTFILES)
    set(LIBRARY_OUTPUT_PATH ${BULLET_PHYSICS_SOURCE_DIR}/lib CACHE PATH "Single output directory for building all libraries.")
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${BULLET_PHYSICS_SOURCE_DIR})
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${BULLET_PHYSICS_SOURCE_DIR})
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${BULLET_PHYSICS_SOURCE_DIR})
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${BULLET_PHYSICS_SOURCE_DIR})
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${BULLET_PHYSICS_SOURCE_DIR})
  else()
    set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR}/lib CACHE PATH "Single output directory for building all libraries.")
  endif()
  option(INTERNAL_CREATE_MSVC_RELATIVE_PATH_PROJECTFILES "Create MSVC projectfiles with relative paths" OFF)
  option(INTERNAL_ADD_POSTFIX_EXECUTABLE_NAMES "Add MSVC postfix for executable names (_Debug)" OFF)
  set(CMAKE_DEBUG_POSTFIX "_Debug" CACHE STRING "Adds a postfix for debug-built libraries.")
  set(CMAKE_MINSIZEREL_POSTFIX "_MinsizeRel" CACHE STRING "Adds a postfix for MinsizeRelease-built libraries.")
  set(CMAKE_RELWITHDEBINFO_POSTFIX "_RelWithDebugInfo" CACHE STRING "Adds a postfix for ReleaseWithDebug-built libraries.")
  if(INTERNAL_CREATE_MSVC_RELATIVE_PATH_PROJECTFILES)
    set(CMAKE_SUPPRESS_REGENERATION ON)
    set(CMAKE_USE_RELATIVE_PATHS ON)
  endif(INTERNAL_CREATE_MSVC_RELATIVE_PATH_PROJECTFILES)
endif()

# Build library
file(GLOB_RECURSE SOURCE "")

add_library(bullet3 STATIC ${SOURCE})
