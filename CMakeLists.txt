
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
include(PlatformToolset)
include(CoreAudioSupport)
include(AAXSupport)
include(VstGuiSupport)
include(UniversalBinary)
include(AddVST3Options)

#-------------------------------------------------------------------------------
# SDK Project
#-------------------------------------------------------------------------------
project(vstsdk)

setupPlatformToolset()

set(ROOT "${CMAKE_CURRENT_SOURCE_DIR}")

# Here you can define where the VST 3 SDK is located
set(SDK_ROOT "${ROOT}")

# Here you can define where the VSTGUI is located
if(SMTG_ADD_VSTGUI)
    set(VSTGUI_ROOT "${ROOT}")
    setupVstGuiSupport()
endif()

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
set(VST_SDK TRUE) # used for Base module which provides only a subset of Base for VST-SDK
add_subdirectory(base)
add_subdirectory(public.sdk)
add_subdirectory(public.sdk/source/vst/interappaudio)

#---Add Wrappers (AU, AAX)-----------------------
if (SMTG_COREAUDIO_SDK_PATH)
    add_subdirectory(public.sdk/source/vst/auwrapper)
endif()

if(SMTG_AAX_SDK_PATH)
    add_subdirectory(public.sdk/source/vst/aaxwrapper)
    set_target_properties(aaxwrapper PROPERTIES ${SDK_IDE_LIBS_FOLDER})
endif()

#-------------------------------------------------------------------------------
# Here is added your own plugins folder location
#-------------------------------------------------------------------------------
set(SDK_IDE_MYPLUGINS_FOLDER FOLDER "MyPlugIns")
if(EXISTS ${SMTG_MYPLUGINS_SRC_PATH})
    set(SMTG_MYPLUGINS_BIN_PATH "${SMTG_MYPLUGINS_SRC_PATH}/build")
    add_subdirectory(${SMTG_MYPLUGINS_SRC_PATH} ${SMTG_MYPLUGINS_BIN_PATH})
endif()


#---Add VST3 examples (plug-ins and hosting)-----
if(SMTG_ADD_VST3_PLUGINS_SAMPLES)
    set(SDK_IDE_PLUGIN_EXAMPLES_FOLDER FOLDER "PlugInExamples")
    add_subdirectory(public.sdk/samples/vst)
    add_subdirectory(public.sdk/source/vst/auwrapper/again)
    add_subdirectory(public.sdk/source/vst/auv3wrapper)
endif()

# Add hosting examples, it includes the validator
set(SDK_IDE_HOSTING_EXAMPLES_FOLDER FOLDER "HostingExamples")
add_subdirectory(public.sdk/samples/vst-hosting)

#-------------------------------------------------------------------------------
# IDE sorting
#-------------------------------------------------------------------------------
set_target_properties(sdk PROPERTIES ${SDK_IDE_LIBS_FOLDER})
set_target_properties(base PROPERTIES ${SDK_IDE_LIBS_FOLDER})

if(SMTG_ADD_VSTGUI)
    set_target_properties(vstgui PROPERTIES ${SDK_IDE_LIBS_FOLDER})
    set_target_properties(vstgui_support PROPERTIES ${SDK_IDE_LIBS_FOLDER})
    set_target_properties(vstgui_uidescription PROPERTIES ${SDK_IDE_LIBS_FOLDER})

    if(TARGET vstgui_standalone)
        set_target_properties(vstgui_standalone PROPERTIES ${SDK_IDE_LIBS_FOLDER})
    endif()
endif()

if(MAC AND XCODE AND SMTG_IOS_DEVELOPMENT_TEAM)
    set_target_properties(base_ios PROPERTIES ${SDK_IDE_LIBS_FOLDER})
endif()
