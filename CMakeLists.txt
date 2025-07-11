cmake_minimum_required(VERSION 3.10)
project(imrad)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_VISIBILITY_PRESET hidden) # linux: don't export all symbols in .so
set(CMAKE_POSITION_INDEPENDENT_CODE ON) # linux: use -fPIC on .a so they can be used by .so

# make use of runtime directory
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set(CMAKE_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}/latest")

if(MSVC)
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
    add_definitions(-D_SCL_SECURE_NO_WARNINGS)
    add_definitions(-D_USE_MATH_DEFINES)
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_DEBUG ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY_RELEASE ${CMAKE_RUNTIME_OUTPUT_DIRECTORY})
else()
  if (${CMAKE_HOST_SYSTEM_NAME} MATCHES "Darwin")
    find_library(MISC_FRAMEWORKS UniformTypeIdentifiers)
    set(CMAKE_OSX_ARCHITECTURES "arm64")
    # Disable ASAN on Debug builds as it may cause issues on arm64
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-frame-pointer")
  elseif(${CMAKE_HOST_SYSTEM_NAME} MATCHES "Linux")
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(MISC REQUIRED gtk+-3.0 glib-2.0)
    set(MISC_FRAMEWORKS "${MISC_STATIC_LIBRARIES}")
  endif()
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-frame-pointer")
    add_definitions(-DGL_SILENCE_DEPRECATION=1)
endif()

# before includes
install(
    DIRECTORY "template/" 
    DESTINATION "template"
)
install(
    DIRECTORY "style/" 
    DESTINATION "style"
)

include(cmake/glfw.cmake)
include(cmake/imgui.cmake)
include(cmake/fa.cmake)
include(cmake/stb.cmake)
include(cmake/freetype.cmake)

include(ExternalProject)
set(NFD_PREFIX 3rdparty/nativefiledialog)
set(NFD_BUILD_TESTS OFF)
ExternalProject_Add(nativefiledialog
  PREFIX ${NFD_PREFIX}
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/${NFD_PREFIX}
  INSTALL_COMMAND ""
  CMAKE_CACHE_ARGS
  -DCMAKE_OSX_ARCHITECTURES:STRING=arm64
)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/${NFD_PREFIX}/src/include)
link_directories(${CMAKE_CURRENT_BINARY_DIR}/${NFD_PREFIX}/src/nativefiledialog-build/src/)

add_subdirectory(src)

file(COPY
        "${CMAKE_CURRENT_SOURCE_DIR}/template"
        DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/bin"
)
file(COPY
        "${CMAKE_CURRENT_SOURCE_DIR}/style"
        DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/bin"
)

