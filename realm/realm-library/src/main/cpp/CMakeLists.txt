cmake_minimum_required(VERSION 3.4.1)

include (UseJava)

#TODO: Find a better to find javah
set(Java_JAVAH_EXECUTABLE "/bin/javah")
set(CMAKE_VERBOSE_MAKEFILE ON)

create_javah(TARGET jni_headers
  CLASSES io.realm.internal.Table io.realm.internal.Util io.realm.internal.TableView
  io.realm.internal.CheckedRow io.realm.internal.SharedGroup io.realm.internal.Group
  io.realm.internal.SubtableSchema io.realm.internal.LinkView io.realm.internal.Util
  io.realm.internal.Version io.realm.internal.UncheckedRow io.realm.internal.TableQuery

  CLASSPATH ${CMAKE_SOURCE_DIR}/../../../build/intermediates/classes/debug/
  OUTPUT_DIR ${CMAKE_SOURCE_DIR}/jni_include
)

# configure import realm core lib
set(debug_SUFFIX "")
if (CMAKE_BUILD_TYPE STREQUAL "Debug")
    set(debug_SUFFIX "-dbg")
endif()
set(core_lib_PATH ${REALM_CORE_DIST_DIR}/librealm-android-${ANDROID_ABI}${debug_SUFFIX}.a)
# Workaround for old core's funny ABI nicknames
if (NOT EXISTS ${core_lib_PATH})
    if (${ANDROID_ABI} STREQUAL "armeabi")
        set(core_lib_PATH ${REALM_CORE_DIST_DIR}/librealm-android-arm.a)
    elseif (${ANDROID_ABI} STREQUAL "armeabi-v7a")
        set(core_lib_PATH ${REALM_CORE_DIST_DIR}/librealm-android-arm-v7a.a)
    elseif (${ANDROID_ABI} STREQUAL "arm64-v8a")
        set(core_lib_PATH ${REALM_CORE_DIST_DIR}/librealm-android-arm64.a)
    else()
        message(FATAL_ERROR "Cannot find core lib file: ${core_lib_PATH}")
    endif()
endif()

add_library(lib_realm_core STATIC IMPORTED )
set_target_properties(lib_realm_core PROPERTIES IMPORTED_LOCATION ${core_lib_PATH})

# build application's shared lib
include_directories(${REALM_CORE_DIST_DIR}/include ${CMAKE_SOURCE_DIR} ${CMAKE_SOURCE_DIR}/jni_include)

# Set compile flags
if (${ANDROID_ABI} STREQUAL "armeabi")
    set(ABI_CXX_FLAGS "-mthumb")
elseif (${ANDROID_ABI} STREQUAL "armeabi-v7a")
    set(ABI_CXX_FLAGS "-mthumb -march=armv7-a -mfloat-abi=softfp -mfpu=vfpv3-d16")
endif()
set(WARNING_CXX_FLAGS "-Wall -Wextra -pedantic -Wno-long-long -Wno-variadic-macros ")
set(REALM_COMMON_CXX_FLAGS " -DREALM_HAVE_CONFIG -DPIC -pthread -fvisibility=hidden -std=gnu++11")
set(CMAKE_CXX_FLAGS_RELEASE "-Os -DNDEBUG")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${REALM_COMMON_CXX_FLAGS} ${WARNING_CXX_FLAGS} ${ABI_CXX_FLAGS}")


# Set link flags
set(REALM_LINKER_FLAGS "-pthread -flto")
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${REALM_LINKER_FLAGS}")

file(GLOB jni_SRC
    "*.cpp"
)
add_library(realm-jni SHARED ${jni_SRC})
add_dependencies(realm-jni jni_headers)
target_link_libraries(realm-jni log lib_realm_core)
