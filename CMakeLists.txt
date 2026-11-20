# zuma - ReXGlue Recompiled Project
#
# SDK boilerplate lives in generated/rexglue.cmake (emitted by scripts/regen.sh).

cmake_minimum_required(VERSION 3.25)
project(zuma VERSION 0.1.4 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# SDK resolution: default to the pinned submodule when REXSDK_DIR isn't set
# (the presets pass $env{REXSDK_DIR}, which lands here as "" when unset).
# Point REXSDK_DIR elsewhere to build against a live SDK checkout instead.
if(NOT REXSDK_DIR AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/rexglue-sdk/CMakeLists.txt")
    set(REXSDK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/rexglue-sdk"
        CACHE PATH "Path to rexglue-sdk source tree" FORCE)
endif()

# Keep the build machine's directory names out of the binaries. Assertions, log
# calls and debug info record source paths; map the tree's root to "." so they
# read "./src/main.cpp" rather than the full path of whoever built it. This has
# to come before the SDK is added below, so that its targets get the flag too.
if(CMAKE_CXX_COMPILER_ID MATCHES "Clang|GNU")
    set(_path_maps "-ffile-prefix-map=${CMAKE_SOURCE_DIR}=.")
    cmake_path(IS_PREFIX CMAKE_SOURCE_DIR "${REXSDK_DIR}" NORMALIZE _sdk_in_tree)
    if(REXSDK_DIR AND NOT _sdk_in_tree)
        list(APPEND _path_maps "-ffile-prefix-map=${REXSDK_DIR}=./rexglue-sdk")
    endif()
    foreach(_map IN LISTS _path_maps)
        if(CMAKE_CXX_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
            set(_map "/clang:${_map}")  # clang-cl spelling
        endif()
        add_compile_options("$<$<COMPILE_LANGUAGE:C,CXX,OBJC,OBJCXX>:${_map}>")
    endforeach()
endif()

if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/generated/rexglue.cmake")
    message(FATAL_ERROR "generated/rexglue.cmake is missing: run scripts/regen.sh first (see README; it also emits this file).")
endif()
include(generated/rexglue.cmake)

# Sources
set(ZUMA_SOURCES
    src/main.cpp
)

if(WIN32)
    # Embed VERSIONINFO (visible in Explorer) via a Windows resource script
    # generated with the project version substituted. An application icon is
    # added here once there is one (src/zuma.ico + an ICON line in zuma.rc.in).
    enable_language(RC)
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/src/zuma.rc.in"
        "${CMAKE_CURRENT_BINARY_DIR}/zuma.rc"
        @ONLY)
    add_executable(zuma WIN32 ${ZUMA_SOURCES} "${CMAKE_CURRENT_BINARY_DIR}/zuma.rc")
else()
    add_executable(zuma ${ZUMA_SOURCES})
endif()

# Scoped to the app target so a version bump only recompiles main.cpp.
target_compile_definitions(zuma PRIVATE ZUMA_VERSION_STRING="${PROJECT_VERSION}")

# GPU_PLUGINS builds + stages rexgpu-xenos (the emulated Xenos GPU, the only
# renderer this port has) next to the exe; without it the runtime has nothing to
# draw with.
rexglue_setup_target(zuma GPU_PLUGINS xenos)

# Same idea for the run-paths the linker records in the shared libraries. The
# SDK's libraries are built outside this build tree, so CMake would write their
# absolute directory into every library that links them. They are only ever
# loaded from beside the executable, so point them there instead.
if(UNIX)
    if(APPLE)
        set(_self_rpath "@loader_path")
    else()
        set(_self_rpath "$ORIGIN")
    endif()
    foreach(_lib rexruntime rexgpu-xenos rexgpu-plume)
        if(TARGET ${_lib})
            set_target_properties(${_lib} PROPERTIES
                BUILD_WITH_INSTALL_RPATH ON
                INSTALL_RPATH "${_self_rpath}"
                INSTALL_RPATH_USE_LINK_PATH OFF)
        endif()
    endforeach()
endif()
