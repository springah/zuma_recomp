# zuma - ReXGlue Recompiled Project
#
# SDK boilerplate lives in generated/rexglue.cmake (emitted by scripts/regen.sh).

cmake_minimum_required(VERSION 3.25)
project(zuma VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# SDK resolution: default to the pinned submodule when REXSDK_DIR isn't set
# (the presets pass $env{REXSDK_DIR}, which lands here as "" when unset).
# Point REXSDK_DIR elsewhere to build against a live SDK checkout instead.
if(NOT REXSDK_DIR AND EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/rexglue-sdk/CMakeLists.txt")
    set(REXSDK_DIR "${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/rexglue-sdk"
        CACHE PATH "Path to rexglue-sdk source tree" FORCE)
endif()

if(NOT EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/generated/rexglue.cmake")
    message(FATAL_ERROR "generated/rexglue.cmake is missing: run scripts/regen.sh first (see README).")
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

rexglue_setup_target(zuma)
