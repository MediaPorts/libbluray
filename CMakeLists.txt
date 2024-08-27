cmake_minimum_required(VERSION 3.20)

# set project name
set(PROJECT_NAME libbluray)

# declare project
project(${PROJECT_NAME})


# configure version
set(BLURAY_VERSION_MAJOR 1)
set(BLURAY_VERSION_MINOR 3)
set(BLURAY_VERSION_MICRO 4)
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libbluray/bluray-version.h.in
  ${CMAKE_CURRENT_SOURCE_DIR}/src/libbluray/bluray-version.h
  @ONLY
)


# defines
ADD_DEFINITIONS(-DJDK_HOME="$ENV{JDK_HOME}")
ADD_DEFINITIONS(-DVERSION="${BLURAY_VERSION_MAJOR}.${BLURAY_VERSION_MINOR}.${BLURAY_VERSION_MICRO}")
ADD_DEFINITIONS(-DHAVE_FCNTL_H)
if(LINUX)
    ADD_DEFINITIONS(-DHAVE_DIRENT_H)
    ADD_DEFINITIONS(-DHAVE_DLFCN_H)
    ADD_DEFINITIONS(-DHAVE_SYS_TIME_H)
endif(LINUX)
if(NOT WIN32)
    ADD_DEFINITIONS(-DHAVE_PTHREAD_H)
endif()
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
    ADD_DEFINITIONS(-DJAVA_ARCH="amd64")
else()
    ADD_DEFINITIONS(-DJAVA_ARCH="i386")
endif()


# classify filters
FILE(GLOB_RECURSE HEADER_FILES
        "contrib/libudfread/*.h"
        "src/file/*.h"
        "src/libbluray/bdj/*.h"
        "src/libbluray/bdj/native*.h"
        "src/libbluray/bdnav/*.h"
        "src/libbluray/decoders/*.h"
        "src/libbluray/disc/*.h"
        "src/libbluray/hdmv/*.h"
        "src/libbluray/*.h"
        "src/util/*.h"
)
FILE(GLOB_RECURSE SOURCE_FILES
        "contrib/libudfread/*.c"
        "src/file/*.c"
        "src/libbluray/bdj/*.c"
        "src/libbluray/bdj/native*.c"
        "src/libbluray/bdnav/*.c"
        "src/libbluray/decoders/*.c"
        "src/libbluray/disc/*.c"
        "src/libbluray/hdmv/*.c"
        "src/libbluray/*.c"
        "src/util/*.c"
)
if(WIN32)
set(REMOVE_PATTERNS
    "*_posix.c"
    "*_darwin.c"
    "*_xdg.c"
    "*_getfsstat.c"
)
elseif(LINUX)
set(REMOVE_PATTERNS
    "*_win32.c"
    "*_darwin.c"
    "*_getfsstat.c"
)
elseif(APPLE)
set(REMOVE_PATTERNS
    "*_win32.c"
    "*_posix.c"
    "*_xdg.c"
    "*_getfsstat.c"
)
elseif(BSD)
set(REMOVE_PATTERNS
    "*_win32.c"
    "*_posix.c"
    "*_xdg.c"
)
else()
set(REMOVE_PATTERNS)
endif()
foreach(PATTERN ${REMOVE_PATTERNS})
    file(GLOB_RECURSE REMOVE_FILES "${PATTERN}")
    list(REMOVE_ITEM SOURCE_FILES ${REMOVE_FILES})
endforeach()
SOURCE_GROUP("Header Files" FILES ${HEADER_FILES})
SOURCE_GROUP("Source Files" FILES ${SOURCE_FILES})


# library
add_library(
    ${PROJECT_NAME}
    ${HEADER_FILES}
    ${SOURCE_FILES}
)


# include
if(WIN32)
target_include_directories(
        ${PROJECT_NAME}
        PRIVATE
        "contrib/libudfread/src"
        "jni"
        "jni/win32"
        "src"
        "src/libbluray"
)
elseif(LINUX)
target_include_directories(
        ${PROJECT_NAME}
        PRIVATE
        "contrib/libudfread/src"
        "jni"
        "jni/linux"
        "src"
        "src/libbluray"
)
elseif(APPLE)
target_include_directories(
        ${PROJECT_NAME}
        PRIVATE
        "contrib/libudfread/src"
        "jni"
        "jni/darwin"
        "src"
        "src/libbluray"
)
elseif(BSD)
    if(CMAKE_SYSTEM_NAME STREQUAL "OpenBSD")
    target_include_directories(
        ${PROJECT_NAME}
        PRIVATE
        "contrib/libudfread/src"
        "jni"
        "jni/openbsd"
        "src"
        "src/libbluray"
    )
    elseif(CMAKE_SYSTEM_NAME STREQUAL "FreeBSD")
    target_include_directories(
        ${PROJECT_NAME}
        PRIVATE
        "contrib/libudfread/src"
        "jni"
        "jni/freebsd"
        "src"
        "src/libbluray"
    )
    elseif(CMAKE_SYSTEM_NAME STREQUAL "NetBSD")
    target_include_directories(
        ${PROJECT_NAME}
        PRIVATE
        "contrib/libudfread/src"
        "jni"
        "jni/netbsd"
        "src"
        "src/libbluray"
    )
    endif()
endif()


