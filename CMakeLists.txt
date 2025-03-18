cmake_minimum_required(VERSION 3.16)

# Projektname und Version
project(gdextension_cpp_example VERSION 1.0 LANGUAGES CXX)

# Setze C++-Standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Check if godot-cpp folder exists, otherwise clone it
if(NOT EXISTS "${CMAKE_SOURCE_DIR}/godot-cpp")
    message(STATUS "godot-cpp folder not found. Cloning from Git...")
    execute_process(
        COMMAND git clone --recurse-submodules https://github.com/godotengine/godot-cpp.git
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    )
endif()

# Godot-C++-Bindings einbinden
add_subdirectory(godot-cpp)

# Quellcode-Verzeichnis hinzufügen
include_directories(src)

# Alle .cpp-Dateien im src-Ordner finden
file(GLOB SOURCES src/*.cpp)

# Bibliothek erstellen
if(APPLE)
    add_library(gdexample SHARED ${SOURCES})
    set_target_properties(gdexample PROPERTIES
        OUTPUT_NAME "libgdexample"
        FRAMEWORK TRUE
    )
elseif(IOS)
    add_library(gdexample STATIC ${SOURCES})
else()
    add_library(gdexample SHARED ${SOURCES})
endif()

# Godot-C++-Bindings mit der Bibliothek verlinken
target_link_libraries(gdexample godot-cpp)