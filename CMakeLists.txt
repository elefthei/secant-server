cmake_minimum_required (VERSION 2.8.7)
project (pistache)
include(CheckCXXCompilerFlag)
include(ExternalProject)

CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
        message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

find_path(ENCLAVE_SGX_SDK PATHS /opt/intel/sgxsdk)
if(ENCLAVE_SGX_SDK_FOUND)
    ExternalProject_Add(enclave
      SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/enclave
      BUILD_IN_SOURCE 1
      CONFIGURE_COMMAND ""
      BUILD_COMMAND make
      INSTALL_COMMAND ""
      PREFIX=${CMAKE_CURRENT_BINARY_DIR}/enclave
    )
endif()

include_directories (${CMAKE_CURRENT_SOURCE_DIR}/include)
file(GLOB HEADER_FILES "include/*.h")
install(FILES ${HEADER_FILES} DESTINATION include/pistache)

file(GLOB SERIALIZER_HEADER_FILES "include/serializer/*.h")
install(FILES ${SERIALIZER_HEADER_FILES} DESTINATION include/pistache/serializer)

add_subdirectory (src)
include_directories (src)

add_subdirectory (app)

find_package(GTest)
if (GTEST_FOUND)
    include_directories(${GTEST_INCLUDE_DIRS})
else()
    ADD_SUBDIRECTORY (googletest-release-1.7.0)
    include_directories(${gtest_SOURCE_DIR}/include ${gtest_SOURCE_DIR})
endif()

enable_testing()
add_subdirectory(tests)
