cmake_minimum_required(VERSION 3.30.5)
project(systest)

if(DEPENDENCY_ON_LOCAL_LIBRARY)
    include(${CMAKE_CURRENT_SOURCE_DIR}/../../../../shadowhook/local_dependency.cmake)
else()
    find_package(shadowhook REQUIRED CONFIG)
endif()

if(${ANDROID_ABI} STREQUAL "arm64-v8a")
    set(ARCH_LINK_FLAGS "-Wl,-z,max-page-size=16384")
elseif(${ANDROID_ABI} STREQUAL "armeabi-v7a")
    set(ARCH_LINK_FLAGS "")
endif()

set(TARGET "shadowhooksystest")
file(GLOB SRC *.c)
add_library(${TARGET} SHARED ${SRC})
target_compile_features(${TARGET} PUBLIC c_std_17)
target_compile_options(${TARGET} PUBLIC -std=c17 -Weverything -Werror -Wno-unused-function)
target_include_directories(${TARGET} PUBLIC .)
target_link_libraries(${TARGET} log shadowhook::shadowhook)

if(DEPENDENCY_ON_LOCAL_LIBRARY)
    target_compile_definitions(${TARGET} PUBLIC -DDEPENDENCY_ON_LOCAL_LIBRARY)
endif()

target_link_options(${TARGET} PUBLIC ${ARCH_LINK_FLAGS})
if(USEASAN)
    target_compile_options(${TARGET} PUBLIC -fsanitize=address -fno-omit-frame-pointer)
    target_link_options(${TARGET} PUBLIC -fsanitize=address)
else()
    target_compile_options(${TARGET} PUBLIC -Oz -ffunction-sections -fdata-sections)
    target_link_options(${TARGET} PUBLIC -Oz -Wl,--exclude-libs,ALL -Wl,--gc-sections -Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/systest.map.txt)
endif()
