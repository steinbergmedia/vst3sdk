
cmake_minimum_required (VERSION 3.4.3)

#-------------------------------------------------------------------------------
# Includes
#-------------------------------------------------------------------------------

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")

include(Global)
include(AddVST3Library)
include(Bundle)
include(ExportedSymbols)
include(PrefixHeader)
include(PlatformIOS)

# do not build VST 2 by default
option(SMTG_CREATE_VST2_VERSION "Use VST2" OFF)

# use by default SMTG_ as prefix for ASSERT,...
option(SMTG_RENAME_ASSERT "Rename ASSERT to SMTG_ASSERT" ON)

#-------------------------------------------------------------------------------
# SDK Project
#-------------------------------------------------------------------------------
set(VST_SDK TRUE)
project(vstsdk)

if (SMTG_RENAME_ASSERT)
	add_compile_options(-DSMTG_RENAME_ASSERT=1)
endif()
	
#------------
if (LINUX)
	# Enable Sample audioHost (based on Jack Audio)
	option(SMTG_ENABLE_USE_OF_JACK "Enable Use of Jack" ON)
	
    option(SMTG_ADD_ADDRESS_SANITIZER_CONFIG "Add AddressSanitizer Config (Linux only)" OFF)
	if(SMTG_ADD_ADDRESS_SANITIZER_CONFIG)
		set(CMAKE_CONFIGURATION_TYPES "${CMAKE_CONFIGURATION_TYPES};ASan")
		add_compile_options($<$<CONFIG:ASan>:-DDEVELOPMENT=1>)
		add_compile_options($<$<CONFIG:ASan>:-fsanitize=address>)
		add_compile_options($<$<CONFIG:ASan>:-DVSTGUI_LIVE_EDITING=1>)
		add_compile_options($<$<CONFIG:ASan>:-g>)
		add_compile_options($<$<CONFIG:ASan>:-O0>)
		set(ASAN_LIBRARY asan)
		link_libraries($<$<CONFIG:ASan>:${ASAN_LIBRARY}>)
	endif()
else()
	# Disable Sample audioHost (based on Jack Audio)
	# not yet tested on Windows and Mac
	option(SMTG_ENABLE_USE_OF_JACK "Enable Use of Jack" OFF)
endif()

#------------
if(UNIX)
	if(XCODE)
		set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LANGUAGE_STANDARD "c++14")
		set(CMAKE_XCODE_ATTRIBUTE_CLANG_CXX_LIBRARY "libc++")
	elseif(APPLE)
		set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -stdlib=libc++")
		link_libraries(c++)
	else()
		set(CMAKE_POSITION_INDEPENDENT_CODE TRUE)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -Wno-multichar")
		link_libraries(stdc++fs pthread dl)
	endif()

#------------
elseif(WIN)
	add_definitions(-D_UNICODE)
	add_compile_options(/fp:fast)
	add_compile_options($<$<CONFIG:Release>:/Oi>)	# Enable Intrinsic Functions (Yes)
	add_compile_options($<$<CONFIG:Release>:/Ot>)	# Favor Size Or Speed (Favor fast code)
	add_compile_options($<$<CONFIG:Release>:/GF>)	# Enable String Pooling
	add_compile_options($<$<CONFIG:Release>:/EHa>)	# Enable C++ Exceptions
	add_compile_options($<$<CONFIG:Release>:/Oy>)	# Omit Frame Pointers
	#add_compile_options($<$<CONFIG:Release>:/Ox>)	# Optimization (/O2: Maximise Speed /0x: Full Optimization)
endif()
#------------

set(ROOT "${CMAKE_CURRENT_SOURCE_DIR}")

# here you can define where the VST 3 SDK is located
set(SDK_ROOT "${ROOT}")

# here you can define where the VSTGUI is located
set(VSTGUI_ROOT "${ROOT}")

include_directories(${ROOT} ${SDK_ROOT})

set(SDK_IDE_LIBS_FOLDER FOLDER "Libraries")
set(SDK_IDE_PLUGIN_EXAMPLES_FOLDER FOLDER "PlugInExamples")
set(SDK_IDE_HOSTING_EXAMPLES_FOLDER FOLDER "HostingExamples")

#-------------------------------------------------------------------------------
# CORE AUDIO SDK Location
#-------------------------------------------------------------------------------
if(MAC AND XCODE)
	if(NOT SMTG_COREAUDIO_SDK_PATH)
		# Check if the CoreAudio SDK is next to the VST3SDK:
		if(EXISTS "${SDK_ROOT}/../CoreAudio/AudioUnits/AUPublic/AUBase/AUBase.cpp")
			set(SMTG_COREAUDIO_SDK_PATH "${SDK_ROOT}/../CoreAudio")
		else()
			if(EXISTS "${SDK_ROOT}/external.apple.coreaudio/AudioUnits/AUPublic/AUBase/AUBase.cpp")
				set(SMTG_COREAUDIO_SDK_PATH "${SDK_ROOT}/external.apple.coreaudio")	
			endif()
		endif()
	else()
		if(NOT IS_ABSOLUTE ${SMTG_COREAUDIO_SDK_PATH})
			get_filename_component(SMTG_COREAUDIO_SDK_PATH "${SDK_ROOT}/${SMTG_COREAUDIO_SDK_PATH}" ABSOLUTE)
		endif()
		if(NOT EXISTS "${SMTG_COREAUDIO_SDK_PATH}/AudioUnits/AUPublic/AUBase/AUBase.cpp")
			message(FATAL_ERROR "SMTG_COREAUDIO_SDK_PATH is set but does not point to an expected location")
		endif()
	endif()
	if(SMTG_COREAUDIO_SDK_PATH)
		message(STATUS "SMTG_COREAUDIO_SDK_PATH is set to : " ${SMTG_COREAUDIO_SDK_PATH})
	else()
		message(STATUS "SMTG_COREAUDIO_SDK_PATH is not set. If you need it, please download the COREAUDIO SDK!")
	endif()
endif()

#-------------------------------------------------------------------------------
# AAX SDK Location
#-------------------------------------------------------------------------------
if(MAC OR WIN)
	if(NOT SMTG_AAX_SDK_PATH)
		# Check if the AAX SDK is next to the VST3SDK:
		if(EXISTS "${SDK_ROOT}/../external.avid.aax/Interfaces/AAX.h")
			set(SMTG_AAX_SDK_PATH "${SDK_ROOT}/../external.avid.aax")
		else()
			if(EXISTS "${SDK_ROOT}/external.avid.aax/Interfaces/AAX.h")
				set(SMTG_AAX_SDK_PATH "${SDK_ROOT}/external.avid.aax")	
			endif()
		endif()
	else()
		if(NOT IS_ABSOLUTE ${SMTG_AAX_SDK_PATH})
			get_filename_component(SMTG_AAX_SDK_PATH "${SDK_ROOT}/${SMTG_AAX_SDK_PATH}" ABSOLUTE)
		endif()
		if(NOT EXISTS "${SMTG_AAX_SDK_PATH}/Interfaces/AAX.h")
			message(FATAL_ERROR "SMTG_AAX_SDK_PATH is set but does not point to an expected location")
		endif()
	endif()
	if(SMTG_AAX_SDK_PATH)
		message(STATUS "SMTG_AAX_SDK_PATH is set to : " ${SMTG_AAX_SDK_PATH})
	else()
		message(STATUS "SMTG_AAX_SDK_PATH is not set. If you need it, please download the AAX SDK!")
	endif()
endif()

#-------------------------------------------------------------------------------
# Projects
#-------------------------------------------------------------------------------

add_subdirectory(base)
add_subdirectory(public.sdk)
add_subdirectory(public.sdk/source/vst/auwrapper)
add_subdirectory(public.sdk/source/vst/auwrapper/again)
add_subdirectory(public.sdk/source/vst/interappaudio)
if(SMTG_AAX_SDK_PATH)
	add_subdirectory(public.sdk/source/vst/aaxwrapper)
endif()

# add VST3 examples (plugins and hosting)
add_subdirectory(public.sdk/samples/vst)
add_subdirectory(public.sdk/samples/vst-hosting)

#-------------------------------------------------------------------------------
# VSTGUI Support Library
#-------------------------------------------------------------------------------
set(VSTGUI_DISABLE_UNITTESTS 1)
set(VSTGUI_DISABLE_STANDALONE 1)
set(VSTGUI_DISABLE_STANDALONE_EXAMPLES 1)
add_subdirectory(vstgui4/vstgui)

add_compile_options($<$<CONFIG:Debug>:-DVSTGUI_LIVE_EDITING=1>)
set(VST3_VSTGUI_SOURCES
	${VSTGUI_ROOT}/vstgui4/vstgui/plugin-bindings/vst3groupcontroller.cpp
	${VSTGUI_ROOT}/vstgui4/vstgui/plugin-bindings/vst3groupcontroller.h
	${VSTGUI_ROOT}/vstgui4/vstgui/plugin-bindings/vst3padcontroller.cpp
	${VSTGUI_ROOT}/vstgui4/vstgui/plugin-bindings/vst3padcontroller.h
	${VSTGUI_ROOT}/vstgui4/vstgui/plugin-bindings/vst3editor.cpp
	${VSTGUI_ROOT}/vstgui4/vstgui/plugin-bindings/vst3editor.h
	${SDK_ROOT}/public.sdk/source/vst/vstguieditor.cpp
)
add_library(vstgui_support STATIC ${VST3_VSTGUI_SOURCES})
target_include_directories(vstgui_support PUBLIC ${VSTGUI_ROOT}/vstgui4)
target_link_libraries(vstgui_support PRIVATE vstgui_uidescription)
if(MAC)
	if(XCODE)
		target_link_libraries(vstgui_support PRIVATE "-framework Cocoa" "-framework OpenGL" "-framework Accelerate" "-framework QuartzCore" "-framework Carbon")
	else()
		find_library(COREFOUNDATION_FRAMEWORK CoreFoundation)
		find_library(COCOA_FRAMEWORK Cocoa)
		find_library(OPENGL_FRAMEWORK OpenGL)
		find_library(ACCELERATE_FRAMEWORK Accelerate)
		find_library(QUARTZCORE_FRAMEWORK QuartzCore)
		find_library(CARBON_FRAMEWORK Carbon)
		target_link_libraries(vstgui_support PRIVATE ${COREFOUNDATION_FRAMEWORK} ${COCOA_FRAMEWORK} ${OPENGL_FRAMEWORK} ${ACCELERATE_FRAMEWORK} ${QUARTZCORE_FRAMEWORK} ${CARBON_FRAMEWORK})
	endif()
endif()

#-------------------------------------------------------------------------------
# IDE sorting
#-------------------------------------------------------------------------------
set_target_properties(vstgui_support PROPERTIES ${SDK_IDE_LIBS_FOLDER})
set_target_properties(sdk PROPERTIES ${SDK_IDE_LIBS_FOLDER})
set_target_properties(base PROPERTIES ${SDK_IDE_LIBS_FOLDER})
set_target_properties(vstgui PROPERTIES ${SDK_IDE_LIBS_FOLDER})
set_target_properties(vstgui_uidescription PROPERTIES ${SDK_IDE_LIBS_FOLDER})
if (TARGET vstgui_standalone)
  set_target_properties(vstgui_standalone PROPERTIES ${SDK_IDE_LIBS_FOLDER})
endif()
if(SMTG_AAX_SDK_PATH)
	set_target_properties(aaxwrapper PROPERTIES ${SDK_IDE_LIBS_FOLDER})
endif()

if(MAC AND XCODE)
	if(IOS_DEVELOPMENT_TEAM)
		set_target_properties(base_ios PROPERTIES ${SDK_IDE_LIBS_FOLDER})
	endif()
endif()
