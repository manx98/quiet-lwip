cmake_minimum_required (VERSION 3.5)
project (Quiet-LWIP)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    add_definitions(-DQUIET_DEBUG)
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} "-g3 -O0")
else()
    if("${CMAKE_C_COMPILER_ID}" STREQUAL "Clang")
      set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} "-Ofast -fsanitize=address -g3")
      set(CMAKE_EXE_LINKER_FLAGS ${CMAKE_EXE_LINKER_FLAGS} "-Ofast -flto")
    else()
      set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} "-O2")
    endif()
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11 -Wpedantic -Wall")
if (NOT APPLE)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_XOPEN_SOURCE=700")
endif()

include_directories(${CMAKE_BINARY_DIR}/include)
include_directories(${CMAKE_SOURCE_DIR}/include)
include_directories(${CMAKE_SOURCE_DIR}/include/lwip)
include_directories(${CMAKE_SOURCE_DIR}/include/lwip/ipv4)

add_subdirectory(3rdparty/fftw-3.3.10)
add_subdirectory(3rdparty/jansson-2.14)
add_subdirectory(3rdparty/libfec-master)
add_subdirectory(3rdparty/libsndfile-1.2.2)
add_subdirectory(3rdparty/portaudio-19.7.0)
add_subdirectory(3rdparty/quiet-dsp-devel)
add_subdirectory(3rdparty/quiet-master)

set(SRCFILES src/driver.c src/util.c)

set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
add_subdirectory(src/lwip)

set(CORE_DEPENDENCIES quiet pthread)

set(SRCFILES ${SRCFILES} src/driver_portaudio.c)
add_definitions(-DQUIET_PORTAUDIO=1)
set(CORE_DEPENDENCIES ${CORE_DEPENDENCIES} portaudio_static)
add_custom_target(quiet-lwip-portaudio-h ALL COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/include/quiet-lwip-portaudio.h ${CMAKE_BINARY_DIR}/include/quiet-lwip-portaudio.h)


add_library(quiet_lwip STATIC ${SRCFILES} $<TARGET_OBJECTS:lwip_sys> $<TARGET_OBJECTS:lwip_core> $<TARGET_OBJECTS:lwip_api> $<TARGET_OBJECTS:lwip_netif> $<TARGET_OBJECTS:lwip_ipv4>)
target_link_libraries(quiet_lwip ${CORE_DEPENDENCIES})

add_custom_target(lwip-h ALL COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/include/lwip/ ${CMAKE_BINARY_DIR}/include/lwip)
add_custom_target(quiet-lwip-h ALL COMMAND ${CMAKE_COMMAND} -E copy ${CMAKE_SOURCE_DIR}/include/quiet-lwip.h ${CMAKE_BINARY_DIR}/include/quiet-lwip.h)

add_subdirectory(examples)
