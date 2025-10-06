cmake_minimum_required(VERSION 3.14)
set(CMAKE_POLICY_VERSION 3.27)      # or 3.30 / 3.31 if that's the CMake you use
set(CMAKE_POLICY_VERSION_MINIMUM 3.5)

project(tremor LANGUAGES C)

option(BUILD_SHARED_LIBS "Build shared libraries" OFF)
option(TREMOR_FETCH_OGG "Fetch libogg if not found in parent build" ON)

# ---- libogg (dependency) -----------------------------------------------------
# If the parent project already provided libogg, we won't fetch it.
if (NOT TARGET Ogg::ogg AND NOT TARGET ogg)
  if (TREMOR_FETCH_OGG)
    include(FetchContent)
    # Prefer release tag; swap if you need a fork
    FetchContent_Declare(
      ogg
      GIT_REPOSITORY https://github.com/xiph/ogg.git
      GIT_TAG        HEAD
    )
    # Silence tests/tools in ogg
    set(BUILD_TESTING OFF CACHE BOOL "" FORCE)
    FetchContent_MakeAvailable(ogg)
  else()
    message(FATAL_ERROR
      "libogg not found and TREMOR_FETCH_OGG=OFF. "
      "Provide Ogg::ogg or ogg target from the parent build.")
  endif()
endif()

# Create a modern namespace alias if upstream exports plain 'ogg'
if (TARGET ogg AND NOT TARGET Ogg::ogg)
  add_library(Ogg::ogg ALIAS ogg)
endif()

# ---- tremor library ----------------------------------------------------------
add_library(tremor
    block.c
    codebook.c
    floor0.c
    floor1.c
    info.c
    mapping0.c
    mdct.c
    registry.c
    res012.c
    sharedbook.c
    synthesis.c
    vorbisfile.c
    window.c
)

# Make tremor headers visible to dependents
# If your headers are under 'include/tremor', prefer that path.
target_include_directories(tremor
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>      # fallback if headers sit at root
    $<INSTALL_INTERFACE:include>
)

# Link libogg and propagate usage to consumers (PUBLIC)
target_link_libraries(tremor PUBLIC Ogg::ogg)

# Good defaults for Android and static consumption
set_target_properties(tremor PROPERTIES
  C_STANDARD 99
  POSITION_INDEPENDENT_CODE ON
)

target_compile_options(tremor PRIVATE
  $<$<C_COMPILER_ID:MSVC>:/wd4244 /wd4267 /wd4996 /wd4005 /wd4114>
  $<$<AND:$<C_COMPILER_ID:Clang>,$<PLATFORM_ID:Windows>>:/wd4244 /wd4267 /wd4996 /wd4005 /wd4114> # clang-cl
)

# Optional: enable ARM asm path used by some forks
if (CMAKE_SYSTEM_PROCESSOR MATCHES "arm" OR CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64")
  target_compile_definitions(tremor PRIVATE ARM_ASSEM)
endif()
