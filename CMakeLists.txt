
cmake_minimum_required (VERSION 3.19)

# Specify the minimum version of the target platform
if(NOT DEFINED ENV{MACOSX_DEPLOYMENT_TARGET})
    set(CMAKE_OSX_DEPLOYMENT_TARGET "10.13" CACHE STRING "macOS deployment target")
    message(STATUS "[SMTG] macOS Deployment Target: ${CMAKE_OSX_DEPLOYMENT_TARGET}")
endif()

#-------------------------------------------------------------------------------
# Options
#-------------------------------------------------------------------------------
# Add VST 3 Plug-ins Samples
option(SMTG_ADD_VST3_PLUGINS_SAMPLES "Add VST 3 Plug-ins Samples to the solution" ON)

# Add and use VSTGUI (enable VST 3 Plug-ins Samples using VSTGUI)
option(SMTG_ADD_VSTGUI "Add VSTGUI Support" ON)

#-------------------------------------------------------------------------------
# Includes
#-------------------------------------------------------------------------------

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules")

include(SMTG_VST3_SDK)

#-------------------------------------------------------------------------------
# SDK Project
#-------------------------------------------------------------------------------
project(vstsdk
    VERSION 3.7.7
    DESCRIPTION "Steinberg VST 3 Software Development Kit"
    HOMEPAGE_URL "https://www.steinberg.net"
)

smtg_setup_platform_toolset()

set(ROOT "${CMAKE_CURRENT_SOURCE_DIR}")

# Here you can define where the VST 3 SDK is located
set(SDK_ROOT "${ROOT}")
set(public_sdk_SOURCE_DIR ${SDK_ROOT}/public.sdk)
set(pluginterfaces_SOURCE_DIR ${SDK_ROOT}/pluginterfaces)

# Disable all VST 3 samples when using SDK as subdirectory
message(STATUS "[SMTG] CMAKE_SOURCE_DIR is set to: ${CMAKE_SOURCE_DIR}")
message(STATUS "[SMTG] CMAKE_CURRENT_LIST_DIR is set to: ${CMAKE_CURRENT_LIST_DIR}")

if(NOT ${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_LIST_DIR})
    message(STATUS "[SMTG] Disable all VST 3 samples")
    set(SMTG_ADD_VST3_PLUGINS_SAMPLES OFF)
    set(SMTG_ADD_VST3_HOSTING_SAMPLES OFF)
    set(SMTG_ADD_MYPLUGINS_SRC_PATH OFF)
else()
    set(SMTG_ADD_MYPLUGINS_SRC_PATH ON)
endif()

# Here you can define where the VSTGUI is located
if(SMTG_ADD_VSTGUI)
    smtg_enable_vstgui_support(VSTGUI_SOURCE_DIR "${ROOT}/vstgui4")
endif(SMTG_ADD_VSTGUI)

include_directories(${ROOT} ${SDK_ROOT})

#-------------------------------------------------------------------------------
# From Here this is optional...

#-------------------------------------------------------------------------------
# CORE AUDIO SDK, AAX SDK, JACK SUPPORT
#-------------------------------------------------------------------------------
setupCoreAudioSupport()
setupAaxSupport()
setupJackSupport()

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

#-------------------------------------------------------------------------------
# Here is added your own plug-ins folder location
# (by default we add the HelloWorld included in my_plugins folder)
#-------------------------------------------------------------------------------
if(SMTG_ADD_MYPLUGINS_SRC_PATH)
    set(SMTG_MYPLUGINS_SRC_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../my_plugins" CACHE PATH "Here you can add Your VST 3 plug-ins folder")

    if(EXISTS ${SMTG_MYPLUGINS_SRC_PATH})
        message(STATUS "[SMTG] SMTG_MYPLUGINS_SRC_PATH is set to: " ${SMTG_MYPLUGINS_SRC_PATH})
    else()
        message(STATUS "[SMTG] SMTG_MYPLUGINS_SRC_PATH is not set. If you want to add your own Plug-ins folder, specify it!")
    endif()

    set(SDK_IDE_MYPLUGINS_FOLDER FOLDER "MyPlugIns")
    if(EXISTS ${SMTG_MYPLUGINS_SRC_PATH})
        set(SMTG_MYPLUGINS_BIN_PATH "${SMTG_MYPLUGINS_SRC_PATH}/build")
        add_subdirectory(${SMTG_MYPLUGINS_SRC_PATH} ${SMTG_MYPLUGINS_BIN_PATH})
    endif()
endif(SMTG_ADD_MYPLUGINS_SRC_PATH)

#---Add VST 3 examples (plug-ins and hosting)-----
if(SMTG_ADD_VST3_PLUGINS_SAMPLES)
    set(SDK_IDE_PLUGIN_EXAMPLES_FOLDER FOLDER "PlugInExamples")
    add_subdirectory(public.sdk/samples/vst)
    add_subdirectory(public.sdk/source/vst/auv3wrapper)
endif(SMTG_ADD_VST3_PLUGINS_SAMPLES)

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

if(SMTG_ADD_VSTGUI)
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
