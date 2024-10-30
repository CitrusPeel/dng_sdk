cmake_minimum_required(VERSION 3.21)
project(dng_sdk)

option(DNG_THREAD_SAFE "Enable thread-safe support in the DNG SDK" ON)
option(DNG_WITH_JPEG "Enable JPEG support in the DNG SDK" ON)
option(DNG_WITH_XMP "Enable XMP support in the DNG SDK" ON)
option(DNG_WITH_JXL "Enable JXL support in the DNG SDK" ON)

#
# DNG SDK
#
set(DNG_SOURCE_DIR ${PROJECT_SOURCE_DIR}/../../source)
file(GLOB DNG_SOURCE ${DNG_SOURCE_DIR}/*.cpp)
add_library(dng STATIC ${DNG_SOURCE})

find_package(ZLIB REQUIRED)
target_link_libraries(dng PRIVATE ZLIB::ZLIB)

target_include_directories(dng PUBLIC $<BUILD_INTERFACE:${DNG_SOURCE_DIR}>)
target_compile_definitions(dng PUBLIC $<$<PLATFORM_ID:Android>:qAndroid=1> $<$<PLATFORM_ID:Darwin>:qMacOS=1>
                                      $<$<PLATFORM_ID:Linux>:qLinux=1> $<$<PLATFORM_ID:Windows>:qWinOS=1>)
if(DNG_THREAD_SAFE)
    find_package(Threads REQUIRED)
    target_link_libraries(dng PRIVATE Threads::Threads)
    target_compile_definitions(dng PUBLIC qDNGThreadSafe=1)
else()
    target_compile_definitions(dng PUBLIC qDNGThreadSafe=0)
endif()

#
# JPEG
#
if(DNG_WITH_JPEG)
    find_package(JPEG REQUIRED)
    target_link_libraries(dng PRIVATE JPEG::JPEG)
    target_compile_definitions(dng PUBLIC qDNGUseLibJPEG=1)
else()
    target_compile_definitions(dng PUBLIC qDNGUseLibJPEG=0)
endif()

#
# XMP
#
if(DNG_WITH_XMP)
    find_package(EXPAT REQUIRED)

    set(XMP_ROOT ${PROJECT_SOURCE_DIR}/../../../xmp)
    file(GLOB XMP_SOURCE ${XMP_ROOT}/toolkit/source/*.cpp)
    list(REMOVE_ITEM XMP_SOURCE ${XMP_ROOT}/toolkit/source/Host_IO-Win.cpp)
    file(GLOB XMP_CORE ${XMP_ROOT}/toolkit/XMPCore/source/*.cpp)
    file(GLOB XMP_COMMON ${XMP_ROOT}/toolkit/XMPCommon/source/*.cpp)
    file(GLOB XMP_ZUID ${XMP_ROOT}/toolkit/third-party/zuid/sources/*.cpp)

    add_library(xmp STATIC ${XMP_SOURCE} ${XMP_CORE} ${XMP_COMMON} ${XMP_ZUID})
    target_link_libraries(xmp PRIVATE EXPAT::EXPAT)
    target_include_directories(
        xmp
        PUBLIC $<BUILD_INTERFACE:${XMP_ROOT}/toolkit/public/include>
        PRIVATE ${XMP_ROOT}/toolkit)
    target_compile_definitions(
        xmp
        PUBLIC $<$<PLATFORM_ID:Darwin>:MAC_ENV> $<$<PLATFORM_ID:Linux>:UNIX_ENV> $<$<PLATFORM_ID:Windows>:WIN_ENV>
        PRIVATE XMP_UNIXBuild
        PRIVATE XMP_StaticBuild
        PRIVATE XMP_COMPONENT_INT_NAMESPACE=AdobeXMPCore_Int)

    target_link_libraries(dng PRIVATE xmp)
    target_compile_definitions(dng PUBLIC qDNGUseXMP=1 qDNGXMPFiles=0 qDNGXMPDocOps=0)
else()
    target_compile_definitions(dng PUBLIC qDNGUseXMP=0)
endif()

#
# JXL
#
if(DNG_WITH_JXL)
    set(JXL_ROOT ${PROJECT_SOURCE_DIR}/../../../libjxl/libjxl)
    set(JPEGXL_ENABLE_DOXYGEN
        false
        CACHE BOOL "JPEGXL_DOXYGEN")
    set(JPEGXL_ENABLE_MANPAGES
        false
        CACHE BOOL "JPEGXL_MANPAGES")
    set(JPEGXL_ENABLE_BENCHMARK
        false
        CACHE BOOL "JPEGXL_BENCHMARK")
    set(JPEGXL_ENABLE_EXAMPLES
        false
        CACHE BOOL "JPEGXL_EXAMPLES")
    set(JPEGXL_ENABLE_TOOLS
        false
        CACHE BOOL "JPEGXL_TOOLS")
    set(JPEGXL_ENABLE_JNI
        false
        CACHE BOOL "JPEGXL_JNI")
    set(JPEGXL_ENABLE_SJPEG
        false
        CACHE BOOL "JPEGXL_SJPEG")
    set(JPEGXL_ENABLE_JPEGLI
        false
        CACHE BOOL "JPEGXL_JPEGLI")
    set(HWY_ENABLE_EXAMPLES
        OFF
        CACHE BOOL "Build examples")
    set(HWY_ENABLE_INSTALL
        OFF
        CACHE BOOL "Install library")
    set(HWY_ENABLE_TESTS
        OFF
        CACHE BOOL "Enable HWY tests")
    set(BUILD_TESTING false)
    add_subdirectory(${JXL_ROOT} ${CMAKE_BINARY_DIR}/libjxl)
    target_link_libraries(dng PUBLIC jxl jxl_threads)
endif()
