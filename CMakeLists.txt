cmake_minimum_required(VERSION 3.0)
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CXX_STANDARD 11)
project (µWebSockets)

option(BUILD_SHARED_LIBS "Build shared libraries." ON)

find_package(OpenSSL REQUIRED)
find_package(ZLIB REQUIRED)

if(NOT LIBUV_INCLUDE_DIR)
    find_path(LIBUV_INCLUDE_DIR uv.h)
endif()
if(NOT LIBUV_LIBRARY)
    find_library(LIBUV_LIBRARY NAMES uv uv1)
endif()

add_library(uWS src/Extensions.cpp src/Group.cpp src/WebSocketImpl.cpp src/Networking.cpp src/Hub.cpp src/Node.cpp src/WebSocket.cpp src/HTTPSocket.cpp src/Socket.cpp)
if(CMAKE_VERSION VERSION_LESS "3.1")
    # CMake 3.0 does not implement "CMAKE_CXX_STANDARD":
    if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
        set(CMAKE_CXX_FLAGS "--std=gnu++11 ${CMAKE_CXX_FLAGS}")
    endif()
endif()
target_include_directories(uWS PUBLIC src)

target_include_directories(uWS PUBLIC ${LIBUV_INCLUDE_DIR})
target_include_directories(uWS PUBLIC ${ZLIB_INCLUDE_DIRS})
target_include_directories(uWS PUBLIC ${OPENSSL_INCLUDE_DIR})
target_link_libraries (uWS LINK_PUBLIC ${LIBUV_LIBRARY})
target_link_libraries (uWS LINK_PUBLIC ${OPENSSL_SSL_LIBRARY})
target_link_libraries (uWS LINK_PUBLIC ${OPENSSL_CRYPTO_LIBRARY})
target_link_libraries (uWS LINK_PUBLIC ${ZLIB_LIBRARY})

if (UNIX)
    target_link_libraries (uWS LINK_PUBLIC pthread)
endif (UNIX)

#============================================================================
# Notes about Windows with a contemporary SDK
# OPENSSL:
#   CONFIGURE_COMMAND perl Configure no-asm VC-WIN64A "--prefix=${EXTERNAL_PREFIX}"
#   BUILD_COMMAND "ms\\do_win64a.bat"
#   COMMAND nmake -f "ms\\ntdll.mak"
#   BUILD_IN_SOURCE 1
#   INSTALL_COMMAND nmake -f "ms\\ntdll.mak" install
# LIBUV:
#   CONFIGURE_COMMAND ""
#   BUILD_IN_SOURCE 1
#   BUILD_COMMAND vcbuild.bat release x64 shared
#   INSTALL_COMMAND ""
#
#   Linking against winsock is required:
#============================================================================
if (WIN32)
    target_link_libraries (uWS LINK_PUBLIC wsock32 ws2_32)
endif (WIN32)


#============================================================================
# Installation (zlib as an example)
#============================================================================
set(INSTALL_BIN_DIR "${CMAKE_INSTALL_PREFIX}/bin" CACHE PATH "Installation directory for executables")
set(INSTALL_LIB_DIR "${CMAKE_INSTALL_PREFIX}/lib" CACHE PATH "Installation directory for libraries")
set(INSTALL_INC_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "Installation directory for headers")
if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    install(TARGETS uWS
        RUNTIME DESTINATION "${INSTALL_BIN_DIR}"
        ARCHIVE DESTINATION "${INSTALL_LIB_DIR}"
        LIBRARY DESTINATION "${INSTALL_LIB_DIR}"
    )
endif()
if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL )
    install(FILES src/Extensions.h src/WebSocketProtocol.h src/Networking.h src/WebSocket.h src/Hub.h src/Group.h src/Node.h src/Socket.h src/HTTPSocket.h src/uWS.h DESTINATION "${INSTALL_INC_DIR}/uWS")
endif()

#============================================================================
# Build examples
#============================================================================
if (NOT SKIP_BUILD_EXAMPLES)
    add_subdirectory(examples)
endif ()
