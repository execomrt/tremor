cmake_minimum_required(VERSION 3.10)
project(tremor LANGUAGES C)

# Optional: allow user to choose static or shared
option(BUILD_SHARED_LIBS "Build shared libraries" OFF)

# Create the library
add_library(tremor
  bitwise.c
  codebook.c
  floor0.c
  floor1.c
  mapping0.c
  mdct.c
  registry.c
  res012.c
  sharedbook.c
  synthesis.c
  vorbisfile.c
  window.c
)

# Include headers
# Include headers
target_include_directories(tremor
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>   # fallback if headers are at root
    $<INSTALL_INTERFACE:include>
)


# Optional: define ARM optimizations
if(CMAKE_SYSTEM_PROCESSOR MATCHES "arm" OR CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64")
  target_compile_definitions(tremor PRIVATE ARM_ASSEM)
endif()
