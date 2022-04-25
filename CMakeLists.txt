cmake_minimum_required(VERSION 3.10)

project("SDL_ttf")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_FLAGS "-D__STDC_CONSTANT_MACROS")
add_compile_options("/EHsc")
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

if (SUPRESS_JSON)
    message("-- IDE configuration setttings are disabled")
else()
    message("-- IDE configuration settings will be written to .vscode/c_cpp_properties.json")
    set(json_filename ${CMAKE_CURRENT_SOURCE_DIR}/.vscode/c_cpp_properties.json)
    set(json_string "{\n\t\"configurations\": [\n\t\t{\n\t\t\t\"name\": \"Win32\",\n\t\t\t\"includePath\": [\n\t\t\t\t\"\${workspaceFolder}/**\",")
endif()

if (SDL2_LIBRARIES AND SDL2_INCLUDE_DIRS)
    message("-- SDL2 is using include dirs ${SDL2_INCLUDE_DIRS} and lib ${SDL2_LIBRARIES}")
else()
    find_package(SDL2)
    if (SDL2_FOUND)
        set(SDL2_LIBRARIES SDL2::SDL2)
        message("-- Found SDL2: ${SLD2_DIR}")
    else()
        message("-- **WARNING** Unable to find SDL2 automatically.  Use -DSDL2_INCLUDE_DIRS and -DSDL2_LIBRARIES to set locations manually")
    endif()
endif()
if (NOT SUPRESS_JSON)
    if (SDL2_FOUND)
        get_target_property(json_sdl2_include SDL2::SDL2 INTERFACE_INCLUDE_DIRECTORIES)
    else()
        set(json_sdl2_include ${SDL2_INCLUDE_DIRS})
    endif()
    list(TRANSFORM json_sdl2_include PREPEND "\n\t\t\t\t\"") 
    list(TRANSFORM json_sdl2_include APPEND "\",")
    string(APPEND json_string ${json_sdl2_include})
endif()

if (NOT FreeType_LIBRARIES)
	set(FreeType_LIBRARIES ${CMAKE_CURRENT_SOURCE_DIR}../../freetype/build/Release/freetype.lib)
endif()
if (NOT FreeType_INCLUDE_DIRS)
	set(FreeType_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}../../freetype/include)
endif()
message("-- FreeType is using include dirs ${FreeType_INCLUDE_DIRS} and lib ${FreeType_LIBRARIES}")


if (NOT SUPRESS_JSON)
    set(json_freetype_include ${FreeType_INCLUDE_DIRS})
    list(TRANSFORM json_freetype_include PREPEND "\n\t\t\t\t\"")
    list(TRANSFORM json_freetype_include APPEND "\",")
    string(APPEND json_string ${json_freetype_include})
endif()

if (NOT SUPRESS_JSON)
    string(APPEND json_string "\n\t\t\t],\n\t\t\t\"defines\": [\n\t\t\t\t\"_DEBUG\",\n\t\t\t\t\"UNICODE\",\n\t\t\t\t\"_UNICODE\"\n\t\t\t],")
    string(APPEND json_string "\n\t\t\t\"windowsSdkVersion\": \"${CMAKE_VS_WINDOWS_TARGET_PLATFORM_VERSION}\",")
    string(APPEND json_string "\n\t\t\t\"compilerPath\": \"${CMAKE_CXX_COMPILER}\",")
    if ("${CMAKE_CXX_STANDARD}" STREQUAL "20")
        string(APPEND json_string "\n\t\t\t\"cStandard\": \"c20\",\n\t\t\t\"cppStandard\": \"c++20\",")
    endif()
    if ("${CMAKE_CXX_STANDARD}" STREQUAL "17")
        string(APPEND json_string "\n\t\t\t\"cStandard\": \"c17\",\n\t\t\t\"cppStandard\": \"c++17\",")
    endif()
    if ("${CMAKE_CXX_STANDARD}" STREQUAL "14")
        string(APPEND json_string "\n\t\t\t\"cStandard\": \"c14\",\n\t\t\t\"cppStandard\": \"c++14\",")
    endif()
    if ("${CMAKE_CXX_STANDARD}" STREQUAL "11")
        string(APPEND json_string "\n\t\t\t\"cStandard\": \"c11\",\n\t\t\t\"cppStandard\": \"c++11\",")
    endif()
    string(APPEND json_string "\n\t\t\t\"intelliSenseMode\": \"windows-msvc-x64\"\n\t\t}\n\t],\n\t\"version\": 4\n}")
    message("-- IDE configuration settings written to .vscode/c_cpp_properties.json")
endif()

file(WRITE ${json_filename} ${json_string})

if (NOT CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    message(STATUS "This project has a top-level one called [${CMAKE_PROJECT_NAME}]")
else()
    message(STATUS "This project is a top-level one")
endif()

add_library(${PROJECT_NAME} STATIC)

target_sources(${PROJECT_NAME} PRIVATE
                    src/SDL_ttf.c
)

target_link_libraries(${PROJECT_NAME} PRIVATE 
                        ${SDL2_LIBRARIES} 
                        ${FreeType_LIBRARIES}
                     )

target_include_directories(${PROJECT_NAME} PUBLIC
                            include
                            ${SDL2_INCLUDE_DIRS}
                            ${FreeType_INCLUDE_DIRS}
                          )

