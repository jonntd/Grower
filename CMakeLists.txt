#-D CMAKE_BUILD_TYPE=Debug
# go to the build directory, cmake from there, specifying the relative path to where CMakeLists.txt is (cmake ..)
cmake_minimum_required(VERSION 2.6)

project(Grower)

SET ( MAYA_PLUGIN_NAME "Grower" )

# Platform specific definitions
IF(WIN32)
    IF(MSVC)
        # To enable /MP, parralel build on MSVC
        OPTION(WIN32_USE_MP "Set to OFF to diable /MP." ON)
        MARK_AS_ADVANCED(WIN32_USE_MP)
        IF(WIN32_USE_MP)
            SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
        ENDIF(WIN32_USE_MP)

        # Other MSVC compilation flags
        ADD_DEFINITIONS(-D_SCL_SECURE_NO_WARNINGS)
        ADD_DEFINITIONS(-D_CRT_SECURE_NO_DEPRECATE)
		ADD_DEFINITIONS(-D NOMINMAX) # prevent windef.h min/max macros from interfering with stl's
		SET (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc")

    ENDIF(MSVC)
ENDIF (WIN32)

# Set the Maya version and architecture (default values)
SET(MAYA_VERSION 2011 CACHE STRING "Maya Version")
SET(MAYA_ARCH x64 CACHE STRING "HW Architecture")

IF(WIN32)
	SET( MAYA_USER_DIR $ENV{USERPROFILE}/Documents/maya CACHE PATH "Maya user home directory" )
	SET( MAYA_ROOT $ENV{PROGRAMFILES}/Autodesk/Maya${MAYA_VERSION} CACHE PATH "Maya installation directory" )
	SET( MAYA_SDK_DEFINITIONS "_AFXDLL;_MBCS;NT_PLUGIN;REQUIRE_IOSTREAM" )

ELSE()
	SET( MAYA_USER_DIR $ENV{HOME}/maya CACHE PATH "Maya user home directory" )
	SET( MAYA_ROOT /usr/autodesk/maya${MAYA_VERSION}-${MAYA_ARCH} )
ENDIF()

FIND_PACKAGE(OpenGL)

SET( MAYA_HEADERS_DIR ${MAYA_ROOT}/include )
SET( MAYA_LIBRARY_DIR ${MAYA_ROOT}/lib )

set( RENDERLIB_INCLUDE_DIR "" CACHE STRING "path to renderlib include folder" )
set( RENDERLIB_LIBRARY_DIR "" CACHE STRING "path to renderlib libray folder" )

SET ( LOCAL_WARNING_FLAGS /W3 )
SET ( LOCAL_RTTI_FLAGS /GR )

SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/../bin)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/../bin)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/../bin)

# add base to include directories
INCLUDE_DIRECTORIES ( ${PUBLIC_INCLUDE_DIRS} ${MAYA_HEADERS_DIR} ${RENDERLIB_INCLUDE_DIR} ${OPENGL_INCLUDE_DIR})
LINK_DIRECTORIES ( ${PUBLIC_LIBRARY_DIRS} ${MAYA_LIBRARY_DIR} ${RENDERLIB_LIBRARY_DIR})

SET( RENDER_LIB RenderLib.lib )
SET( OPENGL_LIB opengl32.lib )
SET ( MAYA_DEFINITIONS "_AFXDLL,_MBCS,NT_PLUGIN,REQUIRE_IOSTREAM" )
SET( MAYASDK_LIBRARIES "Foundation.lib" "OpenMaya.lib" "OpenMayaUI.lib" "OpenMayaAnim.lib" "OpenMayaFX.lib" "OpenMayaRender.lib" )

#specify app sources
FILE(GLOB_RECURSE SOURCE_FILES src/*.c src/*.cpp src/*.h src/*.inl src/*.hpp src/*.glsl src/*.ui)

ADD_LIBRARY ( ${MAYA_PLUGIN_NAME} SHARED ${SOURCE_FILES} )
TARGET_LINK_LIBRARIES( ${MAYA_PLUGIN_NAME} ${MAYASDK_LIBRARIES} ${OPENGL_LIB} ${RENDER_LIB})

SET_TARGET_PROPERTIES( ${MAYA_PLUGIN_NAME} PROPERTIES COMPILE_DEFINITIONS ${MAYA_DEFINITIONS} )
SET_TARGET_PROPERTIES( ${MAYA_PLUGIN_NAME} PROPERTIES OUTPUT_NAME ${MAYA_PLUGIN_NAME} )
SET_TARGET_PROPERTIES( ${MAYA_PLUGIN_NAME} PROPERTIES SUFFIX ".mll" )
SET_TARGET_PROPERTIES( ${MAYA_PLUGIN_NAME} PROPERTIES CLEAN_DIRECT_OUTPUT 1 )
SET_TARGET_PROPERTIES( ${MAYA_PLUGIN_NAME} PROPERTIES LINK_FLAGS "/export:initializePlugin /export:uninitializePlugin" )
