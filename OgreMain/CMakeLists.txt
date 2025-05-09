#-------------------------------------------------------------------
# This file is part of the CMake build system for OGRE-Next
#     (Object-oriented Graphics Rendering Engine)
# For the latest info, see http://www.ogre3d.org/
#
# The contents of this file are placed in the public domain. Feel
# free to make use of it in any way you like.
#-------------------------------------------------------------------

############################################################
# OgreNextMain core library
############################################################

PROJECT(${OGRE_NEXT}Main)

function( add_filtered_h filterName fileString )
	file(GLOB TMP_FILES ${fileString})
	set(HEADER_FILES ${HEADER_FILES} ${TMP_FILES} PARENT_SCOPE)
	source_group(${filterName} FILES ${TMP_FILES})
endfunction( add_filtered_h )

function( add_filtered_src filterName fileString )
	file(GLOB TMP_FILES ${fileString})
	set(SOURCE_FILES ${SOURCE_FILES} ${TMP_FILES} PARENT_SCOPE)
	source_group(${filterName} FILES ${TMP_FILES})
endfunction( add_filtered_src )

function( add_filtered_std relativePath )
	string(REPLACE "/" "\\" filterPart ${relativePath})
	add_filtered_h("Header Files\\${filterPart}" "${CMAKE_CURRENT_SOURCE_DIR}/include/${relativePath}/*.h")
	add_filtered_h("Header Files\\${filterPart}" "${CMAKE_CURRENT_SOURCE_DIR}/include/${relativePath}/*.inl")
	set(HEADER_FILES ${HEADER_FILES} PARENT_SCOPE)
	add_filtered_src("Source Files\\${filterPart}" "${CMAKE_CURRENT_SOURCE_DIR}/src/${relativePath}/*.cpp")
	set(SOURCE_FILES ${SOURCE_FILES} PARENT_SCOPE)
endfunction( add_filtered_std )

function( add_filter_only filterName fileString )
	file(GLOB TMP_FILES ${fileString})
	source_group(${filterName} FILES ${TMP_FILES})
endfunction( add_filter_only )

include(PrecompiledHeader)

# define header and source files for the library
file(GLOB HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h"
	"${CMAKE_CURRENT_SOURCE_DIR}/include/*.inl")
file(GLOB SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/*.inl")

add_filtered_std("Animation")
add_filtered_std("CommandBuffer")
add_filtered_std("Compositor")
add_filtered_std("Compositor/Pass")
add_filtered_std("Compositor/Pass/PassClear")
add_filtered_std("Compositor/Pass/PassCompute")
add_filtered_std("Compositor/Pass/PassDepthCopy")
add_filtered_std("Compositor/Pass/PassIblSpecular")
add_filtered_std("Compositor/Pass/PassMipmap")
add_filtered_std("Compositor/Pass/PassQuad")
add_filtered_std("Compositor/Pass/PassScene")
add_filtered_std("Compositor/Pass/PassShadows")
add_filtered_std("Compositor/Pass/PassStencil")
add_filtered_std("Compositor/Pass/PassTargetBarrier")
add_filtered_std("Compositor/Pass/PassUav")
add_filtered_std("Compositor/Pass/PassWarmUp")
add_filtered_std("Compute")
add_filtered_std("debugbreak")
add_filtered_std("Hash")
add_filtered_std("Math/Array")
add_filtered_std("Math/Simple")
add_filtered_std("Math/Simple/C")
add_filtered_std("ParticleSystem")
add_filtered_std("ogrestd")
add_filtered_std("Vao")

list(APPEND HEADER_FILES ${OGRE_BINARY_DIR}/include/OgreBuildSettings.h
	src/OgreImageResampler.h
	src/OgrePixelConversions.h
	src/OgreSIMDHelper.h)

add_filter_only( "Header Files\\Threading" "${CMAKE_CURRENT_SOURCE_DIR}/include/Threading/*.h" )
add_filter_only( "Source Files\\Threading" "${CMAKE_CURRENT_SOURCE_DIR}/src/Threading/*.cpp" )

# Remove optional header files
list(REMOVE_ITEM HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/include/OgreFreeImageCodec2.h"
	"${CMAKE_CURRENT_SOURCE_DIR}/include/OgreASTCCodec.h"
	"${CMAKE_CURRENT_SOURCE_DIR}/include/OgreDDSCodec2.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/OgrePVRTCCodec.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/OgreETCCodec.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/OgreZip.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/OgreAPKZipArchive.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/OgreSTBICodec.h"
)

# Remove optional source files
list(REMOVE_ITEM SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreFreeImageCodec2.cpp"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/OgreASTCCodec.cpp"
	"${CMAKE_CURRENT_SOURCE_DIR}/src/OgreDDSCodec2.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OgrePVRTCCodec.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreETCCodec.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreZip.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreAPKZipArchive.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreSearchOps.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreSTBICodec.cpp"
)

if (OGRE_SIMD_SSE2)
	add_filtered_std( "Math/Array/SSE2/Single" )
endif()

if (OGRE_SIMD_NEON)
	add_filtered_std( "Math/Array/NEON/Single" )
endif()

if (NOT(OGRE_SIMD_SSE2 OR OGRE_SIMD_NEON))
	add_filtered_std( "Math/Array/C" )
endif ()


# Add platform specific files
if (EMSCRIPTEN)
  include_directories(include/Emscripten)

  file(GLOB PLATFORM_HEADERS ${CMAKE_CURRENT_SOURCE_DIR}/include/Emscripten/*.h)
  file(GLOB PLATFORM_SOURCE_FILES ${CMAKE_CURRENT_SOURCE_DIR}/src/Emscripten/*.cpp src/OgreSearchOps.cpp)

  set(PLATFORM_HEADER_INSTALL "Emscripten")
elseif (WIN32)
  set(THREAD_SOURCE_FILES
      src/Threading/OgreBarrierWin.cpp
	  src/Threading/OgreLightweightMutexWin.cpp
      src/Threading/OgreThreadsWin.cpp
  )
  list(APPEND PLATFORM_SOURCE_FILES src/WIN32/OgreWin32Resources.rc)
  if (WINDOWS_STORE OR WINDOWS_PHONE)
    include_directories("include/WIN32")
    set(PLATFORM_HEADERS
      include/WIN32/OgreComPtr.h
      include/WIN32/OgreConfigDialogImpWinRT.h
      include/WIN32/OgreErrorDialogImpWinRT.h
      include/WIN32/OgreTimerImp.h
    )
    set(PLATFORM_SOURCE_FILES
      src/WIN32/OgreOptimisedUtilDirectXMath.cpp
      src/WIN32/OgreConfigDialogWinRT.cpp
      src/WIN32/OgreErrorDialogWinRT.cpp
      src/WIN32/OgreFileSystemLayer.cpp
      src/WIN32/OgreTimer.cpp
    )  

    set(PLATFORM_HEADER_INSTALL "WIN32")
  else ()
    include_directories("include/WIN32")
    set(PLATFORM_HEADERS
      include/WIN32/OgreComPtr.h
      include/WIN32/OgreConfigDialogImp.h
      include/WIN32/OgreErrorDialogImp.h
      include/WIN32/OgreTimerImp.h
      include/WIN32/OgreMinGWSupport.h
    )
    set(PLATFORM_SOURCE_FILES
      src/WIN32/OgreOptimisedUtilDirectXMath.cpp
      src/WIN32/OgreConfigDialog.cpp
      src/WIN32/OgreErrorDialog.cpp
      src/WIN32/OgreFileSystemLayer.cpp
      src/WIN32/OgreTimer.cpp
      src/WIN32/resource.h
    )  
    set(RESOURCE_FILES
      src/WIN32/resource.h
      src/WIN32/OgreWin32Resources.rc
    )
    source_group(Resources FILES ${RESOURCE_FILES})
    set(PLATFORM_HEADER_INSTALL "WIN32")

    if (MINGW)
      # Older CMake versions do not know how to deal with resource files on MinGW.
      # Newer ones do, but pass along invalid command options to windres.
      # This is a manual fix for the moment.
      if (NOT CMAKE_RC_COMPILER)
        message(SEND_ERROR "Could not find MinGW resource compiler. Please specify path in CMAKE_RC_COMPILER.")
      endif ()
      add_custom_command(OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/OgreWin32Resources.rc.obj
        COMMAND ${CMAKE_RC_COMPILER} -I${CMAKE_CURRENT_SOURCE_DIR}/src/WIN32 
                                     -i${CMAKE_CURRENT_SOURCE_DIR}/src/WIN32/OgreWin32Resources.rc
                                     -o${CMAKE_CURRENT_BINARY_DIR}/OgreWin32Resources.rc.obj
        DEPENDS ${RESOURCE_FILES})
      list(APPEND PLATFORM_SOURCE_FILES ${CMAKE_CURRENT_BINARY_DIR}/OgreWin32Resources.rc.obj)
    else ()
      list(APPEND PLATFORM_SOURCE_FILES src/WIN32/OgreWin32Resources.rc)
    endif ()
  endif ()

elseif (APPLE)
  if (OGRE_BUILD_PLATFORM_APPLE_IOS)
    include_directories("include/iOS")
    file(GLOB PLATFORM_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/include/iOS/*.h")
    file(GLOB PLATFORM_SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/iOS/*.cpp"
      "${CMAKE_CURRENT_SOURCE_DIR}/src/iOS/*.mm"
      src/OgreSearchOps.cpp)

    set(PLATFORM_LIBS "")
    set(PLATFORM_HEADER_INSTALL "iOS")
  else ()
    include_directories("include/OSX")
    
    file(GLOB PLATFORM_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/include/OSX/*.h")
    file(GLOB PLATFORM_SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/OSX/*.cpp"
      "${CMAKE_CURRENT_SOURCE_DIR}/src/OSX/*.mm"
      src/OgreSearchOps.cpp
      include/OSX/ogrelogo.png
    )

    set(RESOURCE_FILES
	  include/OSX/ogrelogo.png
    )
    source_group(Resources FILES ${RESOURCE_FILES})

    set(PLATFORM_LIBS ${Carbon_LIBRARY_FWK} ${Cocoa_LIBRARY_FWK})
    set(PLATFORM_HEADER_INSTALL "OSX")
  endif ()
elseif(ANDROID)
  # required by OgrePlatformInformation.cpp
  include_directories(${ANDROID_NDK}/sources/android/cpufeatures)

  file(GLOB PLATFORM_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/include/Android/*.h")
  file(GLOB PLATFORM_SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/Android/*.cpp"
      src/OgreSearchOps.cpp)

  set(PLATFORM_HEADER_INSTALL "Android")
elseif (UNIX)
	if( NOT OGRE_CONFIG_UNIX_NO_X11 )
		include_directories(include/GLX ${X11_INCLUDE_DIR})

		file(GLOB PLATFORM_HEADERS "${CMAKE_CURRENT_SOURCE_DIR}/include/GLX/*.h"
		  "${CMAKE_CURRENT_SOURCE_DIR}/src/GLX/GLX_backdrop.h")
		file(GLOB PLATFORM_SOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/GLX/*.cpp"
		  src/OgreSearchOps.cpp)

		set(PLATFORM_LIBS ${X11_LIBRARIES} ${X11_Xt_LIB} ${XAW_LIBRARY} ${X11_Xrandr_LIB} xcb pthread)
		set(PLATFORM_HEADER_INSTALL "GLX")
	else()
		file(GLOB PLATFORM_HEADERS
			"${CMAKE_CURRENT_SOURCE_DIR}/include/GLX/OgreTimer.h"
			"${CMAKE_CURRENT_SOURCE_DIR}/include/GLX/OgreErrorDialog.h"
			src/OgreSearchOps.cpp)
		file(GLOB PLATFORM_SOURCE_FILES
			"${CMAKE_CURRENT_SOURCE_DIR}/src/GLX/OgreTimer.cpp"
			"${CMAKE_CURRENT_SOURCE_DIR}/src/GLX/OgreErrorDialog.cpp"
			"${CMAKE_CURRENT_SOURCE_DIR}/src/GLX/OgreFileSystemLayer.cpp"
			src/OgreSearchOps.cpp)

		set(PLATFORM_LIBS pthread)
		set(PLATFORM_HEADER_INSTALL "")
	endif()
	# some platforms require explicit linking to libdl, see if it's available
	find_library(DL_LIBRARY NAMES dl dlsym)
	mark_as_advanced(DL_LIBRARY)
	if (DL_LIBRARY)
		list(APPEND PLATFORM_LIBS dl)
	endif ()
endif()

if (NOT WIN32)
  set(THREAD_SOURCE_FILES
      src/Threading/OgreBarrierPThreads.cpp
	  src/Threading/OgreLightweightMutexPThreads.cpp
      src/Threading/OgreThreadsPThreads.cpp
  )
endif()

list( APPEND THREAD_SOURCE_FILES
	src/Threading/OgreWaitableEvent.cpp
)

if( APPLE )
	list( APPEND THREAD_SOURCE_FILES
		src/Threading/OgreSemaphoreApple.mm
	)
else()
	list( APPEND THREAD_SOURCE_FILES
		src/Threading/OgreSemaphore.cpp
	)
endif()


# Configure threading files
set(THREAD_HEADER_FILES
	include/Threading/OgreBarrier.h
	include/Threading/OgreLightweightMutex.h
	include/Threading/OgreSemaphore.h
	include/Threading/OgreThreadDefines.h
	include/Threading/OgreThreadHeaders.h
	include/Threading/OgreThreads.h
	include/Threading/OgreDefaultWorkQueue.h
	include/Threading/OgreUniformScalableTask.h
	include/Threading/OgreWaitableEvent.h
)
if (OGRE_THREAD_PROVIDER EQUAL 0)
	list(APPEND THREAD_HEADER_FILES
		include/Threading/OgreThreadDefinesNone.h
		include/Threading/OgreDefaultWorkQueueStandard.h
	)
	list(APPEND THREAD_SOURCE_FILES
		src/Threading/OgreDefaultWorkQueueStandard.cpp
	)
elseif (OGRE_THREAD_PROVIDER EQUAL 2)
	list(APPEND THREAD_HEADER_FILES
		include/Threading/OgreThreadDefinesPoco.h
		include/Threading/OgreThreadHeadersPoco.h
		include/Threading/OgreDefaultWorkQueueStandard.h
	)
	list(APPEND THREAD_SOURCE_FILES
		src/Threading/OgreDefaultWorkQueueStandard.cpp
	)
elseif (OGRE_THREAD_PROVIDER EQUAL 3)
	list(APPEND THREAD_HEADER_FILES
		include/Threading/OgreThreadDefinesTBB.h
		include/Threading/OgreThreadHeadersTBB.h
		include/Threading/OgreDefaultWorkQueueTBB.h
	)
	list(APPEND THREAD_SOURCE_FILES
		src/Threading/OgreDefaultWorkQueueTBB.cpp
	)
elseif (OGRE_THREAD_PROVIDER EQUAL 4)
	list(APPEND THREAD_HEADER_FILES
		include/Threading/OgreThreadDefinesSTD.h
		include/Threading/OgreThreadHeadersSTD.h
		include/Threading/OgreDefaultWorkQueueStandard.h
	)
	list(APPEND THREAD_SOURCE_FILES
		src/Threading/OgreDefaultWorkQueueStandard.cpp
	)
endif ()

list(APPEND HEADER_FILES ${THREAD_HEADER_FILES})

# Add needed definitions
add_definitions(-DOGRE_NONCLIENT_BUILD -DFREEIMAGE_LIB -D_MT -D_USRDLL)

set(LIBRARIES ${PLATFORM_LIBS})

if (OGRE_CONFIG_THREADS)
  list(APPEND LIBRARIES ${OGRE_THREAD_LIBRARIES})
endif ()

if (OGRE_PROFILING_PROVIDER STREQUAL "remotery")
    include_directories( ${Remotery_INCLUDE_DIRS} )
    list(APPEND LIBRARIES ${Remotery_LIBRARIES})
endif ()

if (OGRE_CONFIG_ENABLE_FREEIMAGE)
  list(APPEND HEADER_FILES include/OgreFreeImageCodec2.h)
  list(APPEND SOURCE_FILES src/OgreFreeImageCodec2.cpp)
  list(APPEND LIBRARIES    "${FreeImage_LIBRARIES}")
endif()

if (OGRE_CONFIG_ENABLE_STBI)
  list(APPEND HEADER_FILES include/OgreSTBICodec.h)
  list(APPEND SOURCE_FILES src/OgreSTBICodec.cpp)
endif ()

if (OGRE_CONFIG_ENABLE_DDS)
  list(APPEND HEADER_FILES include/OgreDDSCodec2.h)
  list(APPEND SOURCE_FILES src/OgreDDSCodec2.cpp)
endif ()

if (OGRE_CONFIG_ENABLE_PVRTC)
  list(APPEND HEADER_FILES include/OgrePVRTCCodec.h)
  list(APPEND SOURCE_FILES src/OgrePVRTCCodec.cpp)
endif ()

if (OGRE_CONFIG_ENABLE_ETC)
  list(APPEND HEADER_FILES include/OgreETCCodec.h)
  list(APPEND SOURCE_FILES src/OgreETCCodec.cpp)
endif ()

if (OGRE_CONFIG_ENABLE_ASTC)
  list(APPEND HEADER_FILES include/OgreASTCCodec.h)
  list(APPEND SOURCE_FILES src/OgreASTCCodec.cpp)
endif ()

if (OGRE_CONFIG_ENABLE_ZIP)
  list(APPEND HEADER_FILES include/OgreZip.h)
  list(APPEND SOURCE_FILES src/OgreZip.cpp)

  if(ANDROID)
    ADD_DEFINITIONS(-DZZIP_OMIT_CONFIG_H)
  endif()

  list(APPEND LIBRARIES "${ZZip_LIBRARIES}")
  list(APPEND LIBRARIES "${ZLIB_LIBRARIES}")
endif ()

if (OGRE_CONFIG_ENABLE_GLES2_GLSL_OPTIMISER)
  list(APPEND LIBRARIES "${GLSL_Optimizer_LIBRARIES}")
endif ()

set (TARGET_LINK_FLAGS "")

set( BROKEN_FILES_IN_UNITY_BUILD "src/OgreFreeImageCodec2.cpp" )

# setup OgreMain target
if (WINDOWS_STORE OR WINDOWS_PHONE)
	# exclude OgreTimer.cpp from unity builds; causes problem
	list( APPEND BROKEN_FILES_IN_UNITY_BUILD
			"src/WIN32/OgreTimer.cpp" )
else ()
	# exclude these from unity builds; cause problems
	list( APPEND BROKEN_FILES_IN_UNITY_BUILD
			"${CMAKE_CURRENT_SOURCE_DIR}/src/OgreAlignedAllocator.cpp"
	)
	if( UNIX AND NOT OGRE_CONFIG_UNIX_NO_X11 AND NOT ANDROID AND NOT APPLE )
		# These files cause problems on Linux
		list( APPEND BROKEN_FILES_IN_UNITY_BUILD
				"${CMAKE_CURRENT_SOURCE_DIR}/src/GLX/OgreConfigDialog.cpp"
		)
	endif()
endif ()
ogre_add_library(${OGRE_NEXT}Main ${OGRE_LIB_TYPE} ${PREC_HEADER} ${HEADER_FILES} ${SOURCE_FILES} ${PLATFORM_HEADERS} ${PLATFORM_SOURCE_FILES} ${THREAD_HEADER_FILES} ${THREAD_SOURCE_FILES} SEPARATE ${BROKEN_FILES_IN_UNITY_BUILD})

set_target_properties(${OGRE_NEXT}Main PROPERTIES VERSION ${OGRE_SOVERSION} SOVERSION ${OGRE_SOVERSION})

if (NOT CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  if(OGRE_GCC_VERSION VERSION_EQUAL 4.8 OR OGRE_GCC_VERSION VERSION_GREATER 4.8)
    if(CMAKE_SYSTEM_NAME STREQUAL "FreeBSD")
      list(APPEND LIBRARIES "-latomic_ops")
    else()
      list(APPEND LIBRARIES "-latomic")
    endif()
  endif()
endif()

if (APPLE)
  if (OGRE_BUILD_PLATFORM_APPLE_IOS)
    set_target_properties(${OGRE_NEXT}Main PROPERTIES INSTALL_NAME_DIR "OGRE")
  else ()
    set_target_properties(${OGRE_NEXT}Main PROPERTIES
        LINK_FLAGS "-framework IOKit -framework Cocoa -framework Carbon -framework OpenGL -framework CoreVideo"
    )

    set(OGRE_OSX_BUILD_CONFIGURATION "$(PLATFORM_NAME)/$(CONFIGURATION)")
  
	add_custom_command(TARGET ${OGRE_NEXT}Main POST_BUILD
		COMMAND mkdir ARGS -p ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/Ogre.framework/Headers/Threading
        COMMAND ditto 
		${OGRE_SOURCE_DIR}/OgreMain/include/Threading/*.h ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/Ogre.framework/Headers/Threading
		COMMAND mkdir ARGS -p ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/Ogre.framework/Headers/OSX
        COMMAND ditto 
		${OGRE_SOURCE_DIR}/OgreMain/include/OSX/*.h ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/Ogre.framework/Headers/OSX
		COMMAND cd ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/Ogre.framework/Headers
		)

    foreach(HEADER_PATH ${THREAD_HEADER_FILES})
        get_filename_component(HEADER_FILE ${HEADER_PATH} NAME)
        set(FWK_HEADER_PATH ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/Ogre.framework/Headers/${HEADER_FILE})
        add_custom_command(TARGET ${OGRE_NEXT}Main POST_BUILD
            COMMAND rm -f ${FWK_HEADER_PATH}
            )
    endforeach()

    foreach(HEADER_PATH ${PLATFORM_HEADERS})
        get_filename_component(HEADER_FILE ${HEADER_PATH} NAME)
        set(FWK_HEADER_PATH ${OGRE_BINARY_DIR}/lib/${OGRE_OSX_BUILD_CONFIGURATION}/Ogre.framework/Headers/${HEADER_FILE})
        add_custom_command(TARGET ${OGRE_NEXT}Main POST_BUILD
            COMMAND rm -f ${FWK_HEADER_PATH}
            )
    endforeach()

    ogre_config_framework(${OGRE_NEXT}Main)
  endif ()

  # Framework is called 'Ogre'
  if (OGRE_BUILD_LIBS_AS_FRAMEWORKS)
    set_target_properties(${OGRE_NEXT}Main PROPERTIES	OUTPUT_NAME ${OGRE_NEXT})
  endif ()
endif ()
target_link_libraries(${OGRE_NEXT}Main ${LIBRARIES})
if (MINGW)
  # may need winsock htons functions for FreeImage
  target_link_libraries(${OGRE_NEXT}Main ws2_32)
endif ()

# specify a precompiled header to use
use_precompiled_header(${OGRE_NEXT}Main
  "${CMAKE_CURRENT_SOURCE_DIR}/include/OgreStableHeaders.h"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/OgrePrecompiledHeaders.cpp"
)

ogre_config_lib(${OGRE_NEXT}Main TRUE)
foreach(HEADER_FILE ${HEADER_FILES})
	string(REGEX REPLACE "((${CMAKE_CURRENT_SOURCE_DIR}|${OGRE_BINARY_DIR})/)?(include|src)/" "" RELATIVE_HEADER_FILE ${HEADER_FILE})
	string(REGEX MATCH "((.*)/)+" HEADER_SUBDIRECTORY ${RELATIVE_HEADER_FILE})
	install(FILES ${HEADER_FILE} DESTINATION include/${OGRE_NEXT_PREFIX}/${HEADER_SUBDIRECTORY})
endforeach()

install(FILES ${PLATFORM_HEADERS}
  DESTINATION include/${OGRE_NEXT_PREFIX}/${PLATFORM_HEADER_INSTALL}
)
install(FILES ${THREAD_HEADER_FILES}
	DESTINATION include/${OGRE_NEXT_PREFIX}/Threading
)
