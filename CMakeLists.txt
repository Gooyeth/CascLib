cmake_minimum_required(VERSION 2.6)
project(CascLib)

set(HEADER_FILES
    src/CascCommon.h
    src/CascLib.h
    src/CascMndxRoot.h
    src/CascPort.h
    src/common/Common.h
    src/common/FileStream.h
    src/common/ListFile.h
    src/common/Map.h
    src/jenkins/lookup.h
)

set(SRC_FILES
    src/common/Common.cpp
    src/common/Directory.cpp
    src/common/FileStream.cpp
    src/common/ListFile.cpp
    src/common/Map.cpp
    src/jenkins/lookup3.c
    src/CascBuildCfg.cpp
    src/CascCommon.cpp
    src/CascDecompress.cpp
    src/CascDumpData.cpp
    src/CascFindFile.cpp
    src/CascMndxRoot.cpp
    src/CascOpenFile.cpp
    src/CascOpenStorage.cpp
    src/CascReadFile.cpp
)

set(TOMCRYPT_FILES
    src/libtomcrypt/src/hashes/hash_memory.c
    src/libtomcrypt/src/hashes/md5.c
    src/libtomcrypt/src/misc/crypt_argchk.c
    src/libtomcrypt/src/misc/crypt_hash_descriptor.c
    src/libtomcrypt/src/misc/crypt_hash_is_valid.c
    src/libtomcrypt/src/misc/crypt_libc.c
)

set(ZLIB_BZIP2_FILES
    src/zlib/adler32.c
    src/zlib/crc32.c
    src/zlib/inffast.c
    src/zlib/inflate.c
    src/zlib/inftrees.c
    src/zlib/zutil.c
)

set(TEST_SRC_FILES
    test/CascTest.cpp
)

add_definitions(-D_7ZIP_ST -DBZ_STRICT_ANSI)

if(WIN32)
    if(MSVC)
        message(STATUS "Using MSVC")
        add_definitions(-D_7ZIP_ST -DWIN32)
    else()
        message(STATUS "Using mingw")
    endif()
    set(SRC_ADDITIONAL_FILES ${ZLIB_BZIP2_FILES} ${TOMCRYPT_FILES})
    set(LINK_LIBS wininet)
endif()

if(APPLE)
    message(STATUS "Using Mac OS X port")
    set(LINK_LIBS z bz2)
    set(SRC_ADDITIONAL_FILES ${TOMCRYPT_FILES})
endif()

if (${CMAKE_SYSTEM_NAME} STREQUAL Linux)
    message(STATUS "Using Linux port")
    option(WITH_LIBTOMCRYPT "Use system LibTomCrypt library" OFF)
    if(WITH_LIBTOMCRYPT)
        set(LINK_LIBS z bz2 tomcrypt)
    else()
        set(LINK_LIBS z bz2)
        set(SRC_ADDITIONAL_FILES ${TOMCRYPT_FILES})
    endif()
endif()

add_library(casc SHARED ${SRC_FILES} ${SRC_ADDITIONAL_FILES})
target_link_libraries(casc ${LINK_LIBS})

option(WITH_TEST "Compile Test application" OFF)
if(WITH_TEST)
    add_executable(casc_test ${TEST_SRC_FILES})
    target_link_libraries(casc_test casc)
endif()

option(WITH_STATIC "Compile static linked library" OFF)
if(WITH_STATIC)
    add_library(casc_static STATIC ${SRC_FILES} ${SRC_ADDITIONAL_FILES})
    target_link_libraries(casc_static ${LINK_LIBS})
    set_target_properties(casc_static PROPERTIES OUTPUT_NAME casc)
    install(TARGETS casc_static RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib FRAMEWORK DESTINATION /Library/Frameworks)
endif()

if(APPLE)
    set_target_properties(casc PROPERTIES FRAMEWORK true)
    set_target_properties(casc PROPERTIES PUBLIC_HEADER "src/CascLib.h src/CascPort.h")
    set_target_properties(casc PROPERTIES LINK_FLAGS "-framework Carbon")
endif()

if(UNIX)
    set_target_properties(casc PROPERTIES VERSION 1.0.0)
    set_target_properties(casc PROPERTIES SOVERSION 1)
endif()

# On Win32, build CascLib.dll
if(WIN32)
    set_target_properties(casc PROPERTIES OUTPUT_NAME CascLib)
endif()

install(TARGETS casc RUNTIME DESTINATION bin LIBRARY DESTINATION lib ARCHIVE DESTINATION lib FRAMEWORK DESTINATION /Library/Frameworks)
install(FILES src/CascLib.h src/CascPort.h DESTINATION include)
