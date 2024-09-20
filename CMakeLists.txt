cmake_minimum_required(VERSION 3.15)

project(msdfgen VERSION 1.0 LANGUAGES CXX)

if(MSVC)
    set(CMAKE_CXX_FLAGS_DEBUG "/Od /Zi" CACHE STRING "" FORCE)
    set(CMAKE_CXX_FLAGS_DIST "/O2 /DNDEBUG" CACHE STRING "" FORCE)
    set(CMAKE_CXX_FLAGS_RELEASE "/O2 /Ob1 /Zi" CACHE STRING "" FORCE)
    set(${PROJECT_CMAKE_LINKING_FLAGS_TYPE}_DEBUG "/INCREMENTAL:NO /DEBUG:FULL" CACHE STRING "" FORCE)
    set(${PROJECT_CMAKE_LINKING_FLAGS_TYPE}_DIST "/INCREMENTAL:NO /DEBUG:NONE /OPT:REF /OPT:ICF" CACHE STRING "" FORCE)
    set(${PROJECT_CMAKE_LINKING_FLAGS_TYPE}_RELEASE "/INCREMENTAL:NO /DEBUG:FULL /OPT:REF /OPT:ICF" CACHE STRING "" FORCE)
else()
    set(CMAKE_CXX_FLAGS_DEBUG "-O0")
    set(CMAKE_CXX_FLAGS_DIST "-O3 -DNDEBUG")
    set(CMAKE_CXX_FLAGS_RELEASE "-O3 -g")
endif()

file(GLOB_RECURSE MSDFGEN_CORE_HEADERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "core/*.h" "core/*.hpp")
file(GLOB_RECURSE MSDFGEN_CORE_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "core/*.cpp")
file(GLOB_RECURSE MSDFGEN_EXT_HEADERS RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "ext/*.h" "ext/*.hpp")
file(GLOB_RECURSE MSDFGEN_EXT_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} "ext/*.cpp" "lib/*.cpp")

add_library(msdfgen-core "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen.h" ${MSDFGEN_CORE_HEADERS} ${MSDFGEN_CORE_SOURCES})
add_library(msdfgen::msdfgen-core ALIAS msdfgen-core)
set_target_properties(msdfgen-core PROPERTIES PUBLIC_HEADER "${MSDFGEN_CORE_HEADERS}")
set_property(TARGET msdfgen-core PROPERTY MSVC_RUNTIME_LIBRARY "${MSDFGEN_MSVC_RUNTIME}")
target_compile_definitions(msdfgen-core PUBLIC
    MSDFGEN_VERSION=${MSDFGEN_VERSION}
    MSDFGEN_VERSION_MAJOR=${MSDFGEN_VERSION_MAJOR}
    MSDFGEN_VERSION_MINOR=${MSDFGEN_VERSION_MINOR}
    MSDFGEN_VERSION_REVISION=${MSDFGEN_VERSION_REVISION}
    MSDFGEN_COPYRIGHT_YEAR=${MSDFGEN_COPYRIGHT_YEAR}
)
target_include_directories(msdfgen-core INTERFACE
    $<INSTALL_INTERFACE:include/msdfgen>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/>
)
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdfgen-core)

target_compile_features(msdfgen-core PUBLIC cxx_std_11)
target_compile_definitions(msdfgen-core PUBLIC MSDFGEN_USE_CPP11)

target_compile_definitions(msdfgen-core PUBLIC MSDFGEN_PUBLIC=)

add_subdirectory("freetype")

add_library(msdfgen-ext "${CMAKE_CURRENT_SOURCE_DIR}/msdfgen-ext.h" ${MSDFGEN_EXT_HEADERS} ${MSDFGEN_EXT_SOURCES})
add_library(msdfgen::msdfgen-ext ALIAS msdfgen-ext)
set_target_properties(msdfgen-ext PROPERTIES PUBLIC_HEADER "${MSDFGEN_EXT_HEADERS}")
set_property(TARGET msdfgen-ext PROPERTY MSVC_RUNTIME_LIBRARY "${MSDFGEN_MSVC_RUNTIME}")
target_compile_definitions(msdfgen-ext INTERFACE MSDFGEN_EXTENSIONS)
target_compile_definitions(msdfgen-ext PUBLIC MSDFGEN_DISABLE_SVG)
target_compile_definitions(msdfgen-ext PUBLIC MSDFGEN_DISABLE_PNG)
target_link_libraries(msdfgen-ext PRIVATE freetype msdfgen::msdfgen-core)
target_include_directories(msdfgen-ext
    PUBLIC
        $<INSTALL_INTERFACE:include/msdfgen>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
)
set_property(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT msdfgen-ext)

add_library(msdfgen-full INTERFACE)
add_library(msdfgen::msdfgen ALIAS msdfgen-full)
target_link_libraries(msdfgen-full INTERFACE msdfgen::msdfgen-core msdfgen::msdfgen-ext)
