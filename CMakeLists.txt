cmake_minimum_required(VERSION 4.1)
project(GodotPP LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- Configuration ---
set(GODOT_PROJECT_DIR "${CMAKE_SOURCE_DIR}/game")
set(GODOT_VERSION "4.6-stable")
set(GODOT_CACHE_DIR "${CMAKE_SOURCE_DIR}/.godot_bin")

# Detect Godot Binary Path
if(APPLE)
    set(GODOT_EXECUTABLE "${GODOT_CACHE_DIR}/Godot.app/Contents/MacOS/Godot")
    set(GODOT_OUTPUT_DIR "${GODOT_PROJECT_DIR}/bin/macos/arm64")
elseif(WIN32)
    set(GODOT_EXECUTABLE "${GODOT_CACHE_DIR}/Godot_v4/Godot_v${GODOT_VERSION}_windows_win64.exe")
    set(GODOT_OUTPUT_DIR "${GODOT_PROJECT_DIR}/bin/windows")
else()
    set(GODOT_EXECUTABLE "${GODOT_CACHE_DIR}/Godot_v${GODOT_VERSION}_linux.x86_64")
    set(GODOT_OUTPUT_DIR "${GODOT_PROJECT_DIR}/bin/linux")
endif()

add_subdirectory(externals)
add_subdirectory(src)

# Target: setup (Downloads Godot)
add_custom_target(setup
        COMMAND ${CMAKE_COMMAND}
        -DGODOT_VERSION="${GODOT_VERSION}"
        -DGODOT_CACHE_DIR="${GODOT_CACHE_DIR}"
        -P "${CMAKE_SOURCE_DIR}/cmake/DownloadGodot.cmake"
        COMMENT "Checking/Downloading Godot Engine..."
)

# Target: editor (Launches Godot Editor)
add_custom_target(editor
        COMMAND ${GODOT_EXECUTABLE} -e --path "${GODOT_PROJECT_DIR}"
        DEPENDS setup GodotPP
        COMMENT "Launching Godot Editor..."
        USES_TERMINAL
)

# Target: play (Launches the Game)
add_custom_target(play
        COMMAND ${GODOT_EXECUTABLE} --path "${GODOT_PROJECT_DIR}"
        DEPENDS setup GodotPP
        COMMENT "Launching Game..."
        USES_TERMINAL
)

# Target: package (Exports the Game - Simple Example)
# Note: You need an export_presets.cfg in your godot project for this to work
add_custom_target(package
        COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_SOURCE_DIR}/build"
        COMMAND ${GODOT_EXECUTABLE} --headless --export-release "macOS" "${CMAKE_SOURCE_DIR}/build/game.zip" --path "${GODOT_PROJECT_DIR}"
        DEPENDS setup GodotPP
        COMMENT "Packaging Game..."
)