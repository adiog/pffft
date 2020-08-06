cmake_minimum_required(VERSION 3.0)

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

option(HAVE_FFTW ON)
option(USE_SIMD ON)

if(UNIX AND NOT APPLE)
  set(LINUX ON)
endif()

if(LINUX)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra")
  set(CMAKE_C_FLAGS_DEBUG "-g -O3")
  set(CMAKE_C_FLAGS_RELEASE "-O3")

  set(LIBRARIES m)

  if(HAVE_FFTW)
    add_definitions(-DHAVE_FFTW)
    set(LIBRARIES "${LIBRARIES}" "fftw3f")
  endif()

  if(USE_SIMD)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -msse -mfpmath=sse")
  else()
    add_definitions(-DPFFFT_SIMD_DISABLE)
  endif()
endif()

if(APPLE)
  add_definitions(-DHAVE_VECLIB)
  if(HAVE_FFTW)
    add_definitions(-DHAVE_FFTW)
    set(LIBRARIES "${LIBRARIES}" "fftw3f")
  endif()
  set(LIBRARIES "${LIBRARIES}" "-framework Accelerate")
endif()

if(WIN32)
  # cl /Ox -D_USE_MATH_DEFINES /arch:SSE test_pffft.c pffft.c fftpack.c
endif()

add_library(
  pffft
    ${CMAKE_CURRENT_SOURCE_DIR}/fftpack.c
    ${CMAKE_CURRENT_SOURCE_DIR}/fftpack.h
    ${CMAKE_CURRENT_SOURCE_DIR}/pffft.c
    ${CMAKE_CURRENT_SOURCE_DIR}/pffft.h
)

target_link_libraries(pffft ${LIBRARIES})

add_executable(test_pffft ${CMAKE_CURRENT_SOURCE_DIR}/test_pffft.c)
target_link_libraries(test_pffft pffft)
