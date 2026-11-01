cmake_minimum_required(VERSION 3.31)

project(craft)

FILE(GLOB SOURCE_FILES src/*.c)

add_executable(
    craft
    ${SOURCE_FILES}
    deps/lodepng/lodepng.c
    deps/noise/noise.c
    deps/tinycthread/tinycthread.c)

add_definitions(-std=c99 -O3 -DPREFIX=\"${CMAKE_INSTALL_PREFIX}\")

include_directories(deps/lodepng)
include_directories(deps/noise)
include_directories(deps/tinycthread)

if(SERVER)
	add_library(world SHARED deps/noise/noise.c src/world.c)
endif(SERVER)

install_programs(/bin FILES craft)
install_files(/share/craft/shaders FILES shaders/block_fragment.glsl shaders/block_vertex.glsl
        shaders/line_fragment.glsl shaders/line_vertex.glsl shaders/sky_fragment.glsl
	shaders/sky_vertex.glsl shaders/text_fragment.glsl shaders/text_vertex.glsl)
install_files(/share/craft/textures FILES textures/font.png textures/sign.png
	textures/sky.png textures/texture.png)

if(MINGW)
    set(CMAKE_LIBRARY_PATH ${CMAKE_LIBRARY_PATH}
        "C:/Program Files/CURL/lib" "C:/Program Files (x86)/CURL/lib")
    set(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH}
        "C:/Program Files/CURL/include" "C:/Program Files (x86)/CURL/include")
endif()

find_package(CURL REQUIRED)
include_directories(${CURL_INCLUDE_DIR})

if(APPLE)
    target_link_libraries(craft glfw
        ${GLFW_LIBRARIES} ${CURL_LIBRARIES})
endif()

if(UNIX)
    find_package(SQLite3 REQUIRED)
    find_package(OpenGL REQUIRED)
    find_package(glfw3 REQUIRED)
    find_package(GLEW REQUIRED)
    target_link_libraries(craft ${SQLite3_LIBRARIES})
    target_link_libraries(craft ${OPENGL_LIBRARIES})
    target_link_libraries(craft ${GLEW_LIBRARIES})
    target_link_libraries(craft glfw m pthread
        ${GLFW_LIBRARIES} ${CURL_LIBRARIES})
endif()

if(MINGW)
    target_link_libraries(craft ws2_32.lib glfw
        ${GLFW_LIBRARIES} ${CURL_LIBRARIES})
endif()
