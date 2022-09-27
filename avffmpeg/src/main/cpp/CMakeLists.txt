cmake_minimum_required(VERSION 3.4.1)

## 设置GCC编译器的编译选项
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ffunction-sections -fdata-sections -fvisibility=hidden")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ffunction-sections -fdata-sections -fvisibility=hidden")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--gc-sections,-Bsymbolic-functions")

## 设置关键路径
set(ffmpeg_include ${CMAKE_SOURCE_DIR}/include)
set(ffmpeg_so_path ${CMAKE_SOURCE_DIR}/${ANDROID_ABI})

## x264
set(x264_a ${ffmpeg_so_path}/libx264.a)
add_library(x264 STATIC IMPORTED)
set_target_properties(x264 PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m;dl"
        IMPORTED_LOCATION ${x264_a}
        )

## fdkaac
set(fdkaac_a ${ffmpeg_so_path}/libfdk-aac.a)
add_library(fdkaac STATIC IMPORTED)
set_target_properties(fdkaac PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C;CXX"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m"
        IMPORTED_LOCATION ${fdkaac_a}
        )


## avutil
set(avutil_a ${ffmpeg_so_path}/libavutil.a)
add_library(avutil STATIC IMPORTED)
set_target_properties(avutil PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m"
        IMPORTED_LOCATION ${avutil_a}
        )

## postproc
set(postproc_a ${ffmpeg_so_path}/libpostproc.a)
add_library(postproc STATIC IMPORTED)
set_target_properties(postproc PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m;avutil"
        IMPORTED_LOCATION ${postproc_a}
        )

## swscale
set(swscale_a ${ffmpeg_so_path}/libswscale.a)
add_library(swscale STATIC IMPORTED)
set_target_properties(swscale PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m;avutil"
        IMPORTED_LOCATION ${swscale_a}
        )

## swresample
set(swresample_a ${ffmpeg_so_path}/libswresample.a)
add_library(swresample STATIC IMPORTED)
set_target_properties(swresample PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m;avutil"
        IMPORTED_LOCATION ${swresample_a}
        )

## avcodec
set(avcodec_a ${ffmpeg_so_path}/libavcodec.a)
add_library(avcodec STATIC IMPORTED)
set_target_properties(avcodec PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m;z;swresample;avutil;x264;fdkaac"
        IMPORTED_LOCATION ${avcodec_a}
        )

## avformat
set(avformat_a ${ffmpeg_so_path}/libavformat.a)
add_library(avformat STATIC IMPORTED)
set_target_properties(avformat PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m;z;avcodec;swresample;avutil;x264;fdkaac"
        IMPORTED_LOCATION ${avformat_a}
        )

## avfilter
set(avfilter_a ${ffmpeg_so_path}/libavfilter.a)
add_library(avfilter STATIC IMPORTED)
set_target_properties(avfilter PROPERTIES
        IMPORTED_LINK_INTERFACE_LANGUAGES "C"
        IMPORTED_LINK_INTERFACE_LIBRARIES "m;swscale;postproc;avformat;avcodec;swresample;avutil;x264;fdkaac"
        IMPORTED_LOCATION ${avfilter_a}
        )

file(GLOB_RECURSE SOURCE_FILES ./*.cpp ./*.c ./*.cc ./*.h)
include_directories(${ffmpeg_include})

add_library( # Sets the name of the library.

        avffmpeg
        # Sets the library as a shared library.
        SHARED
        # Provides a relative path to your source file(s).
        #        FfmpegMuxer.cpp
        #        FfmpegMuxer.h
        #        ffmpegmuxer_jni.cpp
        ${SOURCE_FILES})

find_library( # Sets the name of the path variable.
        log-lib

        # Specifies the name of the NDK library that
        # you want CMake to locate.
        log)

target_link_libraries( # Specifies the target library.
        avffmpeg
        -Wl,--start-group
        avutil
        fdkaac
        postproc
        swresample
        swscale
        x264
        avcodec
        avformat
        avfilter
        -Wl,--end-group
        # Links the target library to the log library
        # included in the NDK.
        ${log-lib})





