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

#set(lib_src_DIR ${CMAKE_SOURCE_DIR}/../../../libsrc)
#set(lib_build_DIR $ENV{HOME}/tmp)
#file(MAKE_DIRECTORY ${lib_build_DIR})


#add_subdirectory(${lib_src_DIR}/gmath
#                 ${lib_build_DIR}/gmath)
#add_subdirectory(${lib_src_DIR}/gperf
#                 ${lib_build_DIR}/gperf)

# configure import libs
set(distribution_DIR ${CMAKE_SOURCE_DIR}/../../../distribution)
#LINK_DIRECTORIES(${distribution_DIR}/gmath/lib/${ANDROID_ABI})

add_library(lib_realm_core STATIC IMPORTED )
set_target_properties(lib_realm_core PROPERTIES IMPORTED_LOCATION
    ${distribution_DIR}/realm_core/librealm-android-${ANDROID_ABI}.a)

# build application's shared lib
include_directories(${distribution_DIR}/realm_core/include ${CMAKE_SOURCE_DIR} ${CMAKE_SOURCE_DIR}/jni_include)

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
