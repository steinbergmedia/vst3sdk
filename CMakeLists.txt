
cmake_minimum_required (VERSION 3.19.0)

# Specify the minimum version of the target platform
if(NOT DEFINED ENV{MACOSX_DEPLOYMENT_TARGET})
    set(CMAKE_OSX_DEPLOYMENT_TARGET "10.13" CACHE STRING "macOS deployment target")
endif()

# Global options which can bet set on command line e.g.: cmake -DSMTG_ENABLE_VST3_PLUGIN_EXAMPLES=OFF ...
option(SMTG_ENABLE_VST3_PLUGIN_EXAMPLES "Enable VST 3 Plug-in Examples" ON)
option(SMTG_ENABLE_VST3_HOSTING_EXAMPLES "Enable VST 3 Hosting Examples" ON)
option(SMTG_ENABLE_VSTGUI_SUPPORT "Enable VSTGUI Support" ON)

#-------------------------------------------------------------------------------
# Includes
#-------------------------------------------------------------------------------

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")

include(SMTG_VST3_SDK)

#-------------------------------------------------------------------------------
# SDK Project
#-------------------------------------------------------------------------------
project(vstsdk
    VERSION 3.7.11
    DESCRIPTION "Steinberg VST 3 Software Development Kit"
    HOMEPAGE_URL "https://www.steinberg.net"
)

smtg_setup_platform_toolset()
smtg_setup_symbol_visibility()

set(ROOT "${CMAKE_CURRENT_SOURCE_DIR}")

# Set the location of the VST 3 SDK
set(SDK_ROOT "${ROOT}")
set(public_sdk_SOURCE_DIR ${SDK_ROOT}/public.sdk)
set(pluginterfaces_SOURCE_DIR ${SDK_ROOT}/pluginterfaces)

if(SMTG_ENABLE_VSTGUI_SUPPORT)
    smtg_enable_vstgui_support(VSTGUI_SOURCE_DIR "${ROOT}/vstgui4")
endif()

include_directories(${ROOT} ${SDK_ROOT})

#-------------------------------------------------------------------------------
# From Here this is optional...

#-------------------------------------------------------------------------------
# CORE AUDIO SDK, AAX SDK
#-------------------------------------------------------------------------------
setupCoreAudioSupport()
setupAaxSupport()
include(SMTG_FindJack)

#-------------------------------------------------------------------------------
# Projects
#-------------------------------------------------------------------------------
set(SDK_IDE_LIBS_FOLDER FOLDER "Libraries")

#---Add base libraries---------------------------
set(VST_SDK TRUE) # used for pluginterfaces and public.sdk modules which provides only a subset of them for VST-SDK
add_subdirectory(pluginterfaces)
add_subdirectory(base)
add_subdirectory(public.sdk)
add_subdirectory(public.sdk/source/vst/interappaudio)

#---Add Wrappers (AU, AAX)-----------------------
if(NOT "${SMTG_COREAUDIO_SDK_PATH}" STREQUAL "")
    add_subdirectory(public.sdk/source/vst/auwrapper)
endif()

if(NOT "${SMTG_AAX_SDK_PATH}" STREQUAL "")
    add_subdirectory(public.sdk/source/vst/aaxwrapper)
    set_target_properties(aaxwrapper
        PROPERTIES
            ${SDK_IDE_LIBS_FOLDER}
    )
endif()

# Add hosting examples, it includes the validator (must be done before any plug-ins to support running the validator after building)
set(SDK_IDE_HOSTING_EXAMPLES_FOLDER FOLDER "HostingExamples")
add_subdirectory(public.sdk/samples/vst-hosting)

# Add utilities
set(SDK_IDE_UTILITIES_FOLDER FOLDER "Utilities")
add_subdirectory(public.sdk/samples/vst-utilities)

#---Add VST 3 examples (plug-ins and hosting)-----
if(SMTG_ENABLE_VST3_PLUGIN_EXAMPLES)
    set(SDK_IDE_PLUGIN_EXAMPLES_FOLDER FOLDER "PlugInExamples")
    add_subdirectory(public.sdk/samples/vst)
    add_subdirectory(public.sdk/source/vst/auv3wrapper)
endif()

#-------------------------------------------------------------------------------
# IDE sorting
#-------------------------------------------------------------------------------
include(SMTG_CustomModuleTarget)

set_property(
    TARGET
        sdk
        sdk_common
        sdk_hosting
        base
        pluginterfaces
        cmake_modules
        cmake_VST_modules
    PROPERTY 
        ${SDK_IDE_LIBS_FOLDER}
)

if(TARGET base_ios)
    set_property(
        TARGET
            base_ios
            pluginterfaces_ios
        PROPERTY 
            ${SDK_IDE_LIBS_FOLDER}
    )
endif()

if(SMTG_ENABLE_VSTGUI_SUPPORT)
    set_property(
        TARGET
            vstgui
            vstgui_support
            vstgui_uidescription
        PROPERTY 
            ${SDK_IDE_LIBS_FOLDER}
    )
 
    if(TARGET vstgui_standalone)
        set_target_properties(vstgui_standalone
            PROPERTIES
                ${SDK_IDE_LIBS_FOLDER}
        )
    endif()
endif()

#-------------------------------------------------------------------------------
# macOS & iOS collection targets 
#-------------------------------------------------------------------------------
option(SMTG_VSTSDK_GENERATE_MACOS_IOS_COLLECTION_TARGETS "Create macOS and iOS collection targets" OFF)

if(SMTG_VSTSDK_GENERATE_MACOS_IOS_COLLECTION_TARGETS)

	function(get_all_targets var)
		set(targets)
		get_all_targets_recursive(targets ${CMAKE_CURRENT_SOURCE_DIR})
		set(${var} ${targets} PARENT_SCOPE)
	endfunction()

	macro(get_all_targets_recursive targets dir)
		get_property(subdirectories DIRECTORY ${dir} PROPERTY SUBDIRECTORIES)
		foreach(subdir ${subdirectories})
			get_all_targets_recursive(${targets} ${subdir})
		endforeach()

		get_property(current_targets DIRECTORY ${dir} PROPERTY BUILDSYSTEM_TARGETS)
		list(APPEND ${targets} ${current_targets})
	endmacro()

	get_all_targets(all_targets)

	add_custom_target(iOS_Targets)
	add_custom_target(macOS_Targets)

	foreach(target_name ${all_targets})
		if(${target_name} MATCHES "ios")
			add_dependencies(iOS_Targets ${target_name})
		else()
			add_dependencies(macOS_Targets ${target_name})
		endif()
	endforeach()

endif()

