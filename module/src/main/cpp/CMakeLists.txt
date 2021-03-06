cmake_minimum_required(VERSION 3.4.1)

if (NOT DEFINED MODULE_NAME)
    message(FATAL_ERROR "MODULE_NAME is not set")
endif ()

add_definitions(-DRIRU_MODULE)
add_definitions(-DRIRU_MODULE_API_VERSION=${RIRU_MODULE_API_VERSION})
add_definitions(-DRIRU_MODULE_VERSION=${RIRU_MODULE_VERSION})
add_definitions(-DRIRU_MODULE_VERSION_NAME=${RIRU_MODULE_VERSION_NAME})

message("Build type: ${CMAKE_BUILD_TYPE}")

set(CMAKE_CXX_STANDARD 11)

set(LINKER_FLAGS "-ffixed-x18 -Wl,--hash-style=both")
set(C_FLAGS "-Werror=format -fdata-sections -ffunction-sections -fno-exceptions")

if (CMAKE_BUILD_TYPE STREQUAL "Release")
    set(C_FLAGS "${C_FLAGS} -O2 -fvisibility=hidden -fvisibility-inlines-hidden")
    set(LINKER_FLAGS "${LINKER_FLAGS} -Wl,-exclude-libs,ALL -Wl,--gc-sections")
else ()
    add_definitions(-DDEBUG)
    set(C_FLAGS "${C_FLAGS} -O0")
endif ()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${C_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${C_FLAGS}")

set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${LINKER_FLAGS}")
set(CMAKE_MODULE_LINKER_FLAGS "${CMAKE_MODULE_LINKER_FLAGS} ${LINKER_FLAGS}")

find_package(riru REQUIRED CONFIG)
find_package(nativehelper REQUIRED CONFIG)

add_library(${MODULE_NAME} SHARED
        main.cpp
        rirud.cpp
        socket.cpp
        misc.cpp
        dex_file.cpp
        system_server.cpp
        binder_hook.cpp
        bridge_service.cpp
        android.cpp
        plt.c
        manager_process.cpp
        settings_process.cpp
        tinynew.cpp)
target_link_libraries(${MODULE_NAME} log riru::riru nativehelper::nativehelper_header_only)
set_target_properties(${MODULE_NAME} PROPERTIES LINK_FLAGS_RELEASE -s)

add_custom_command(TARGET ${MODULE_NAME} POST_BUILD
        COMMAND ${CMAKE_STRIP} --remove-section=.comment "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/lib${MODULE_NAME}.so")

add_executable(libstarter.so
        starter.cpp misc.cpp android.cpp)
target_link_libraries(libstarter.so log riru::riru)
set_target_properties(libstarter.so PROPERTIES LINK_FLAGS_RELEASE -s)

add_custom_command(TARGET libstarter.so POST_BUILD
        COMMAND ${CMAKE_STRIP} --remove-section=.comment "libstarter.so")