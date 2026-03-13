cmake_minimum_required(VERSION 3.16)
project(gdextension_cpp_example VERSION 1.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(Callgrind ON)

if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang" OR "${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
# using Clang or GCC
    if(Callgrind STREQUAL "ON")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -pedantic -Werror -O2")
        #set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O2")
    else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -pedantic -Werror -O2 -fsanitize=undefined -fno-sanitize-recover=all -fsanitize=float-divide-by-zero -fsanitize=float-cast-overflow -fno-sanitize=null -fno-sanitize=alignment -fsanitize=thread -Wno-error=tsan")
    endif()

# using Intel C++
    elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Intel")

# using Visual Studio C++
    elseif("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
        if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
        string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    else()
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
    endif()
endif()

# use pthreads if unix or unix-alike (linux)
if(UNIX)
  set(THREADS_PREFER_PTHREAD_FLAG ON)
  find_package(Threads REQUIRED)
endif()

## Godot this should only work on linux
# Check if godot-cpp folder exists, otherwise clone it
if(NOT EXISTS "${CMAKE_SOURCE_DIR}/godot-cpp")
    message(STATUS "godot-cpp folder not found. Cloning from Git...")
    execute_process(
        COMMAND git clone --recurse-submodules https://github.com/godotengine/godot-cpp.git
        WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
    )
endif()

# Godot-C++-bindings
# Temporarily remove -pedantic for godot-cpp
string(REPLACE "-pedantic" "" CMAKE_CXX_FLAGS_NO_PEDANTIC "${CMAKE_CXX_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS_NO_PEDANTIC}")
add_subdirectory(godot-cpp)
# Restore the original CMAKE_CXX_FLAGS
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS_NO_PEDANTIC} -pedantic")


INCLUDE_DIRECTORIES(${CMAKE_BINARY_DIR})
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

FILE(GLOB CPP_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp")
FILE(GLOB HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.hpp")
FILE(GLOB HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h")

# create and link the library
add_library(gdexample SHARED ${CPP_FILES})
target_link_libraries(gdexample godot-cpp)