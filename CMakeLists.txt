cmake_minimum_required(VERSION 2.8.8)

project(SOLID)

set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/CMakeModules;${CMAKE_MODULE_PATH}")

set(CMAKE_DEBUG_POSTFIX  "_d")

set(OUTPUT_BINDIR ${PROJECT_BINARY_DIR}/bin)
make_directory(${OUTPUT_BINDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${OUTPUT_BINDIR})

set(OUTPUT_LIBDIR ${PROJECT_BINARY_DIR}/lib)
make_directory(${OUTPUT_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${OUTPUT_LIBDIR})
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${OUTPUT_LIBDIR})


# Dynamic vs Static Linking
option(DYNAMIC_SOLID "Set to ON to build SOLID for dynamic linking.  Use OFF for static." ON)

if(NOT DYNAMIC_SOLID)
  add_definitions(-DSOLID_STATIC)
endif(NOT DYNAMIC_SOLID)

include_directories(
	${PROJECT_SOURCE_DIR}/include
)

if (WIN32)
  if (CMAKE_SIZEOF_VOID_P EQUAL 8)
	set(ARCH "x64")		
  else()
    set(ARCH "x86")	
  endif()	
  if (MSVC)
    if (MSVC90)
      set(PLATFORM "msvc90")
    elseif(MSVC10)
      set(PLATFORM "msvc100")
    elseif(MSVC11) 	
      set(PLATFORM "msvc110")
    elseif(MSVC12)
      set(PLATFORM "msvc120")
    else()
      set(PLATFORM "msvcxxx")
    endif()
  else()
    set(PLATFORM "unknown")
  endif ()

  set(GLUT_ROOT_PATH "${PROJECT_SOURCE_DIR}/../3rdPartyDeps/${ARCH}/${PLATFORM}")
endif(WIN32)


find_package(GLUT)
find_package(OpenGL)


if(MSVC) 
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /D_SCL_SECURE_NO_WARNINGS /D_CRT_SECURE_NO_WARNINGS")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /D_SCL_SECURE_NO_WARNINGS /D_CRT_SECURE_NO_WARNINGS")
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /GS-") 
  set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} /GS-") 
  set(CMAKE_EXE_LINKER_FLAGS_RELWITHDEBINFO "/INCREMENTAL:NO")
  set(CMAKE_SHARED_LINKER_FLAGS_RELWITHDEBINFO "/INCREMENTAL:NO")
  if(CMAKE_CL_64)
  else(CMAKE_CL_64)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} /arch:SSE")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /arch:SSE")
  endif(CMAKE_CL_64)
endif(MSVC)

if(UNIX)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -ffast-math -msse")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -ffast-math -msse")
  if (DYNAMIC_SOLID)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
  endif(DYNAMIC_SOLID)
  set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -funroll-loops")
  set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -DNDEBUG") 
  set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} -DNDEBUG") 
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -lrt")
  if(GNUC)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -mfpmath=sse")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -mfpmath=sse")
  endif(GNUC)
endif(UNIX)

if(UNIX AND NOT WIN32 AND NOT APPLE)
  if(CMAKE_SIZEOF_VOID_P MATCHES "8")
      set(LIB_POSTFIX "64" CACHE STRING "suffix for 32/64 dir placement")
      mark_as_advanced(LIB_POSTFIX)
  endif(CMAKE_SIZEOF_VOID_P MATCHES "8")
endif(UNIX AND NOT WIN32 AND NOT APPLE)
if(NOT DEFINED LIB_POSTFIX)
    set(LIB_POSTFIX "")
endif(NOT DEFINED LIB_POSTFIX)

add_subdirectory(src)
add_subdirectory(examples)
#add_subdirectory(doc)
