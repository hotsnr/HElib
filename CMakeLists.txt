cmake_minimum_required(VERSION 3.2)
project(fhe)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

option(ENABLE_TEST "Build the test files" OFF)
option(ENABLE_THREADS "To use multi-thread. Make Sure the NTL is built with NTL_THREADS=on" OFF)
option(BUILD_SHARED "To build shared objects" OFF)
option(BUILD_AES "To build homomorphic AES" OFF)

## Might modify the following path to NTL according to your environment
set(NTL_HEADER /usr/local/include) 
set(NTL_LIB /usr/local/lib) 
include_directories(${NTL_HEADER})
link_directories(${NTL_LIB})
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
## Use -std=c++11 as dedeault. 
set(CMAKE_CXX_STANDARD 11)
## On old platform, might change to -std=c++0x
#set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")

## To use multi-threads.
## Make sure the NTL is built with NTL_THREADS=on
## For NTL's verison older than 7.0, should turn FHE_THREADS off.
if (ENABLE_THREADS)
set(FHE_THREADS ON) 
add_definitions(-DFHE_THREADS)
set(RUN_LIB ntl gmp pthread)
else (ENABLE_THREADS)
set(FHE_THREADS OFF) 
set(RUN_LIB ntl gmp)
endif (ENABLE_THREADS)
set(TEST_RUN_LIB fhe ${RUN_LIB})

file(GLOB HELIB_SRC_FILES "src/*.cpp")
file(GLOB TEST_FILES "src/Test_*.cpp")

## Remove test files from the source file list.
foreach(prg ${TEST_FILES})
	list(FILTER HELIB_SRC_FILES EXCLUDE REGEX ${prg})
endforeach(prg)

if (BUILD_SHARED)
add_library(fhe SHARED ${HELIB_SRC_FILES})
else (BUILD_SHARED)
add_library(fhe STATIC ${HELIB_SRC_FILES})
endif (BUILD_SHARED)
target_link_libraries(fhe ${RUN_LIB})
target_include_directories(fhe PUBLIC  ${CMAKE_CURRENT_SOURCE_DIR}/src)

if (BUILD_AES)
    add_executable(TEST_AES_exe src/aes/simpleAES.cpp src/aes/homAES.cpp src/aes/Test_AES.cpp)
    target_link_libraries(TEST_AES_exe ${TEST_RUN_LIB})
endif(BUILD_AES)

if (ENABLE_TEST)
    foreach(TEST_FILE ${TEST_FILES})
	get_filename_component(TEST_NAME ${TEST_FILE} NAME)
	add_executable(${TEST_NAME}_exe ${TEST_FILE})
	target_link_libraries(${TEST_NAME}_exe ${TEST_RUN_LIB})
    endforeach(TEST_FILE)
endif(ENABLE_TEST)

