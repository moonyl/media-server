cmake_minimum_required(VERSION 3.17)
project(media_server)

set(CMAKE_CXX_STANDARD 17)

include(ExternalProject)
set(live555_prefix_dir ${CMAKE_BINARY_DIR}/live555-prefix)
set(live555_prefix_src_dir ${live555_prefix_dir}/src/live555)
ExternalProject_Add(live555
        URL http://www.live555.com/liveMedia/public/live555-latest.tar.gz
        SOURCE_DIR ${live555_prefix_src_dir}
        BINARY_DIR ${live555_prefix_src_dir}
        CONFIGURE_COMMAND ./genMakefiles linux-64bit
        INSTALL_COMMAND make install PREFIX=<INSTALL_DIR>
        )

#ExternalProject_Get_Property(live555 BINARY_DIR)
add_library(liveMedia STATIC IMPORTED)
set_target_properties(liveMedia PROPERTIES
        IMPORTED_LOCATION ${live555_prefix_dir}/lib/libliveMedia.a
        INTERFACE_INCLUDE_DIRECTORIES ${live555_prefix_dir}/include/liveMedia)

add_library(groupsock STATIC IMPORTED)
set_target_properties(groupsock PROPERTIES
        IMPORTED_LOCATION ${live555_prefix_dir}/lib/libgroupsock.a
        INTERFACE_INCLUDE_DIRECTORIES ${live555_prefix_dir}/include/groupsock)

add_library(UsageEnvironment STATIC IMPORTED)
set_target_properties(UsageEnvironment PROPERTIES
        IMPORTED_LOCATION ${live555_prefix_dir}/lib/libUsageEnvironment.a
        INTERFACE_INCLUDE_DIRECTORIES ${live555_prefix_dir}/include/UsageEnvironment)

add_library(BasicUsageEnvironment STATIC IMPORTED)
set_target_properties(BasicUsageEnvironment PROPERTIES
        IMPORTED_LOCATION ${live555_prefix_dir}/lib/libBasicUsageEnvironment.a
        INTERFACE_INCLUDE_DIRECTORIES ${live555_prefix_dir}/include/BasicUsageEnvironment)

find_package(OpenSSL REQUIRED)
add_executable(media_server main.cpp DynamicRTSPServer.cpp)
add_dependencies(media_server live555)
target_link_libraries(media_server
        liveMedia BasicUsageEnvironment UsageEnvironment groupsock
        OpenSSL::SSL OpenSSL::Crypto)