
cmake_minimum_required (VERSION 3.19)

#-------------------------------------------------------------------------------
# Options
#-------------------------------------------------------------------------------
# Add VST3 Plug-ins Samples
option(SMTG_ADD_VST3_PLUGINS_SAMPLES "Add VST3 Plug-ins Samples to the solution" ON)

# Add and use VSTGUI (enable VST3 Plug-ins Samples using VSTGUI)
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
    VERSION 3.7.5
    DESCRIPTION "Steinberg VST 3 Software Development Kit"
    HOMEPAGE_URL "https://www.steinberg.net"
)

smtg_setup_platform_toolset()

set(ROOT "${CMAKE_CURRENT_SOURCE_DIR}")

# Here you can define where the VST 3 SDK is located
set(SDK_ROOT "${ROOT}")
set(public_sdk_SOURCE_DIR ${SDK_ROOT}/public.sdk)
set(pluginterfaces_SOURCE_DIR ${SDK_ROOT}/pluginterfaces)

# Disable all VST3 samples when using SDK as subdirectory
message(STATUS "[SMTG] CMAKE_SOURCE_DIR is set to: ${CMAKE_SOURCE_DIR}")
message(STATUS "[SMTG] CMAKE_CURRENT_LIST_DIR is set to: ${CMAKE_CURRENT_LIST_DIR}")

if(NOT ${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_CURRENT_LIST_DIR})
    message(STATUS "[SMTG] Disable all VST3 samples")
    set(SMTG_ADD_VST3_PLUGINS_SAMPLES OFF)
    set(SMTG_ADD_VST3_HOSTING_SAMPLES OFF)
    set(SMTG_ADD_MYPLUGINS_SRC_PATH OFF)
else()
    set(SMTG_ADD_MYPLUGINS_SRC_PATH ON)
endif()

# Here you can define where the VSTGUI is located
if(SMTG_ADD_VSTGUI)
    set(SMTG_VSTGUI_ROOT "${ROOT}")
    smtg_enable_vstgui_support()
endif(SMTG_ADD_VSTGUI)

include_directories(${ROOT} ${SDK_ROOT})

#-------------------------------------------------------------------------------
# From Here this is optional...

#-------------------------------------------------------------------------------
# CORE AUDIO SDK, AAX SDK
#-------------------------------------------------------------------------------
setupCoreAudioSupport()
setupAaxSupport()

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
if(SMTG_COREAUDIO_SDK_PATH)
    add_subdirectory(public.sdk/source/vst/auwrapper)
endif(SMTG_COREAUDIO_SDK_PATH)

if(SMTG_AAX_SDK_PATH)
    add_subdirectory(public.sdk/source/vst/aaxwrapper)
    set_target_properties(aaxwrapper
        PROPERTIES
            ${SDK_IDE_LIBS_FOLDER}
    )
endif(SMTG_AAX_SDK_PATH)

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
    set(SMTG_MYPLUGINS_SRC_PATH "${CMAKE_CURRENT_SOURCE_DIR}/../my_plugins" CACHE PATH "Here you can add Your VST3 Plug-ins folder")

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

#---Add VST3 examples (plug-ins and hosting)-----
if(SMTG_ADD_VST3_PLUGINS_SAMPLES)
    set(SDK_IDE_PLUGIN_EXAMPLES_FOLDER FOLDER "PlugInExamples")
    add_subdirectory(public.sdk/samples/vst)
    add_subdirectory(public.sdk/source/vst/auwrapper/again)
    add_subdirectory(public.sdk/source/vst/auv3wrapper)
endif(SMTG_ADD_VST3_PLUGINS_SAMPLES)

#-------------------------------------------------------------------------------
# IDE sorting
#-------------------------------------------------------------------------------
set_target_properties(sdk
    PROPERTIES
        ${SDK_IDE_LIBS_FOLDER}
)
set_target_properties(sdk_common
    PROPERTIES
        ${SDK_IDE_LIBS_FOLDER}
)
set_target_properties(sdk_hosting
    PROPERTIES
        ${SDK_IDE_LIBS_FOLDER}
)
set_target_properties(base
    PROPERTIES
        ${SDK_IDE_LIBS_FOLDER}
)
set_target_properties(pluginterfaces
    PROPERTIES
        ${SDK_IDE_LIBS_FOLDER}
)
if(TARGET base_ios)
    set_target_properties(base_ios
        PROPERTIES
            ${SDK_IDE_LIBS_FOLDER}
    )
    set_target_properties(pluginterfaces_ios
        PROPERTIES
            ${SDK_IDE_LIBS_FOLDER}
    )
endif(TARGET base_ios)

if(SMTG_ADD_VSTGUI)
    set_target_properties(vstgui
        PROPERTIES
            ${SDK_IDE_LIBS_FOLDER}
    )
    set_target_properties(vstgui_support
        PROPERTIES
            ${SDK_IDE_LIBS_FOLDER}
    )
    set_target_properties(vstgui_uidescription
        PROPERTIES
            ${SDK_IDE_LIBS_FOLDER}
    )

    if(TARGET vstgui_standalone)
        set_target_properties(vstgui_standalone
            PROPERTIES
                ${SDK_IDE_LIBS_FOLDER}
        )
    endif(TARGET vstgui_standalone)
endif(SMTG_ADD_VSTGUI)

include(SMTG_CustomModuleTarget)
set_target_properties(cmake_modules PROPERTIES ${SDK_IDE_LIBS_FOLDER})
set_target_properties(cmake_VST_modules PROPERTIES ${SDK_IDE_LIBS_FOLDER})
