cmake_minimum_required(VERSION 3.15)
project(bcryptXP LANGUAGES C CXX)

add_definitions(-D_WINDLL -DUNICODE -D_UNICODE)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

# set pch config
set(PCH_HEADER "pch.h")
set(PCH_SOURCE "pch.cpp")

# Add sources
set(SOURCES
    ${PCH_SOURCE}
    dllmain.cpp
)

# Add headers
set(HEADERS
    bcrypt.h
    framework.h
    ${PCH_HEADER}
)

# Building dll
add_library(bcryptXP SHARED ${SOURCES} ${HEADERS})

# define export symbols
set_target_properties(bcryptXP PROPERTIES LINK_FLAGS "/DEF:${CMAKE_CURRENT_SOURCE_DIR}/lib.def")

# pch settings (MSVC)
if (MSVC)
    target_precompile_headers(bcryptXP PRIVATE ${PCH_HEADER})
endif()

# Add mbedtls deps (via FetchContent)
include(FetchContent)
FetchContent_Declare(
    mbedtls
    GIT_REPOSITORY https://github.com/Mbed-TLS/mbedtls.git
    GIT_TAG        v3.6.3
)
set(ENABLE_TESTING OFF CACHE BOOL "Disable mbedtls tests" FORCE)
set(ENABLE_PROGRAMS OFF CACHE BOOL "Disable mbedtls programs" FORCE)
FetchContent_MakeAvailable(mbedtls)

file(REMOVE
  ${mbedtls_SOURCE_DIR}/3rdparty/CMakeLists.txt
)

# Patch mbedtls/CMakeLists.txt: remove `add_subdirectory(3rdparty)`
set(MBEDTLS_TOP_CMAKELISTS "${mbedtls_SOURCE_DIR}/CMakeLists.txt")
file(READ "${MBEDTLS_TOP_CMAKELISTS}" MBEDTLS_CMAKELIST_CONTENTS)

string(REPLACE "add_subdirectory(3rdparty)" ""
               MBEDTLS_CMAKELIST_CONTENTS_PATCHED "${MBEDTLS_CMAKELIST_CONTENTS}")

file(WRITE "${MBEDTLS_TOP_CMAKELISTS}" "${MBEDTLS_CMAKELIST_CONTENTS_PATCHED}")

message(STATUS "Patched out add_subdirectory(3rdparty) from MbedTLS CMakeLists.txt")

# Link mbedcrypto lib
target_link_libraries(bcryptXP PRIVATE mbedcrypto)

# include path
target_include_directories(bcryptXP PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${mbedtls_SOURCE_DIR}/include
)
