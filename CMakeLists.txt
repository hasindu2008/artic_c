# Disable in-source builds to prevent source tree corruption.
if(" ${CMAKE_SOURCE_DIR}" STREQUAL " ${CMAKE_BINARY_DIR}")
  message(FATAL_ERROR "FATAL: In-source builds are not allowed.
       You should create a separate directory for build files.")
endif()

# choose the architecture 
set(DEPLOY_PLATFORM "unknown" CACHE STRING "choose the instruction set architecture")
set_property(CACHE DEPLOY_PLATFORM PROPERTY STRINGS unknown x86 arm64-v8a armeabi-v7a)
IF (${DEPLOY_PLATFORM} STREQUAL "unknown")
	message(FATAL_ERROR "choose the DEPLOY_PLATFORM") 
	return() # This is to stop proceeding further and to stop opencv getting set to the default ANDROID_ABI
ENDIF()


# Set minimum cmake version + cmake settings
cmake_minimum_required (VERSION 3.9)
cmake_policy(SET CMP0048 OLD)

# Specify the project, and set the use of c++ and c
project(Artic_c LANGUAGES CXX C)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -fPIC -rdynamic -Wall -O2")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -rdynamic -Wall -O2 -std=c++11 ")


include_directories(${PROJECT_SOURCE_DIR})
include_directories(${PROJECT_SOURCE_DIR}/src)
include_directories(${PROJECT_SOURCE_DIR}/3rdparty)

file(COPY test_files/nCoV-2019.bed
		DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY test_files/samtools.bam
		DESTINATION ${CMAKE_CURRENT_BINARY_DIR})

set(trim src/trim.cpp)
set(common src/common.cpp)


# Build a shared lib
add_executable(artic_c ${common} ${trim})

# htslib dependency
set (htslib_shared "${PROJECT_SOURCE_DIR}/3rdparty/${DEPLOY_PLATFORM}/libhts.so")

#
IF (${DEPLOY_PLATFORM} STREQUAL "x86")
	target_link_libraries(artic_c -lz -ldl -lm -lpthread ${htslib_shared})
ELSE()
	target_link_libraries(artic_c -lz -ldl -lm ${htslib_shared})
ENDIF()
