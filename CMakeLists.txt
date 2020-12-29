cmake_minimum_required (VERSION 2.8)
cmake_policy(VERSION 2.8)

project (OpenGLPractice)

IF(NOT CMAKE_BUILD_TYPE)
    SET(CMAKE_BUILD_TYPE Debug CACHE STRING "Choose the type of build (Debug or Release)" FORCE)
ENDIF(NOT CMAKE_BUILD_TYPE)


if(WIN32)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)
endif(WIN32)

link_directories(${CMAKE_SOURCE_DIR}/lib)

list(APPEND CMAKE_CXX_FLAGS "-std=c++11")


# link external
add_subdirectory(external/glfw-3.3)
include_directories
(
        external/glfw-3.3/include
        external/glm
)
set(LIBS glfw)
set(LIBS ${LIBS} opengl32)
set(CHAPTERS polygonal)


configure_file(configuration/root_directory.h.in configuration/root_directory.h)
include_directories(${CMAKE_BINARY_DIR}/configuration)

# first create relevant static libraries
add_library(STB_IMAGE "src/stb_image.cpp")
set(LIBS ${LIBS} STB_IMAGE)

add_library(GLAD "src/glad.c")
set(LIBS ${LIBS} GLAD)

macro(makeLink src dest target)
    add_custom_command(TARGET ${target} POST_BUILD COMMAND ${CMAKE_COMMAND} -E create_symlink ${src} ${dest}  DEPENDS  ${dest} COMMENT "mklink ${src} -> ${dest}")
endmacro()

foreach(CHAPTER ${CHAPTERS})
    file(GLOB SOURCE
            "src/${CHAPTER}/*.h"
            "src/${CHAPTER}/*.cpp"
            "src/${CHAPTER}/*.glsl"
            )
    set(NAME "${CHAPTER}")
    add_executable(${NAME} ${SOURCE})
    target_link_libraries(${NAME} ${LIBS})
    set_target_properties(${NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin/${CHAPTER}")
    # copy shader files to build directory
    file(GLOB SHADERS
            "src/${CHAPTER}/*.glsl"
            )
    foreach(SHADER ${SHADERS})
    # configure_file(${SHADER} "test")
    add_custom_command(TARGET ${NAME} PRE_BUILD COMMAND ${CMAKE_COMMAND} -E copy ${SHADER} $<TARGET_FILE_DIR:${NAME}>)
    endforeach(SHADER)
    # if compiling for visual studio, also use configure file for each project (specifically to set up working directory)
    if(MSVC)
        configure_file(${CMAKE_SOURCE_DIR}/configuration/visualstudio.vcxproj.user.in ${CMAKE_CURRENT_BINARY_DIR}/${NAME}.vcxproj.user @ONLY)
    endif(MSVC)
endforeach(CHAPTER)

include_directories(${CMAKE_SOURCE_DIR}/includes)