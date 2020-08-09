cmake_minimum_required(VERSION 3.17)
project(media_server)

set(CMAKE_CXX_STANDARD 17)

set (live555_prefix_dir ${CMAKE_CURRENT_BINARY_DIR}/live555-build)
configure_file(CMakeLists.txt.in live555-download/CMakeLists.txt)
execute_process(COMMAND ${CMAKE_COMMAND} -G "${CMAKE_GENERATOR}" .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/live555-download )
if(result)
    message(FATAL_ERROR "CMake step for googletest failed: ${result}")
endif()
execute_process(COMMAND ${CMAKE_COMMAND} --build .
        RESULT_VARIABLE result
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/live555-download )
if(result)
    message(FATAL_ERROR "Build step for live555 failed: ${result}")
endif()

#ExternalProject_Get_Property(live555 BINARY_DIR)
add_library(liveMedia STATIC IMPORTED)
add_dependencies(liveMedia live555)
set_target_properties(liveMedia PROPERTIES
        IMPORTED_LOCATION ${live555_prefix_dir}/lib/libliveMedia.a
        INTERFACE_INCLUDE_DIRECTORIES ${live555_prefix_dir}/include/liveMedia)

add_library(groupsock STATIC IMPORTED)
add_dependencies(groupsock live555)
set_target_properties(groupsock PROPERTIES
        IMPORTED_LOCATION ${live555_prefix_dir}/lib/libgroupsock.a
        INTERFACE_INCLUDE_DIRECTORIES ${live555_prefix_dir}/include/groupsock)

add_library(UsageEnvironment STATIC IMPORTED)
add_dependencies(UsageEnvironment live555)
set_target_properties(UsageEnvironment PROPERTIES
        IMPORTED_LOCATION ${live555_prefix_dir}/lib/libUsageEnvironment.a
        INTERFACE_INCLUDE_DIRECTORIES ${live555_prefix_dir}/include/UsageEnvironment)

add_library(BasicUsageEnvironment STATIC IMPORTED)
add_dependencies(BasicUsageEnvironment live555)
set_target_properties(BasicUsageEnvironment PROPERTIES
        IMPORTED_LOCATION ${live555_prefix_dir}/lib/libBasicUsageEnvironment.a
        INTERFACE_INCLUDE_DIRECTORIES ${live555_prefix_dir}/include/BasicUsageEnvironment)

find_package(OpenSSL REQUIRED)
add_executable(media_server main.cpp DynamicRTSPServer.cpp)
#add_dependencies(media_server live555)
target_link_libraries(media_server
        liveMedia BasicUsageEnvironment UsageEnvironment groupsock
        OpenSSL::SSL OpenSSL::Crypto)