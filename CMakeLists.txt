# Lowest CMake version tried. Lower versions might work.
cmake_minimum_required(VERSION 3.5)

include(GNUInstallDirs)

# Create release build, if not specified.
# (from https://blog.kitware.com/cmake-and-the-default-build-type/)
set(default_build_type "RelWithDebInfo")
if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE STRING "Build type" FORCE)
  set_property(
    CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif ()

set(
  headers
  TinyJS.h
  TinyJS_Functions.h
  TinyJS_MathFunctions.h)

set(
  sources
  TinyJS.cpp
  TinyJS_Functions.cpp
  TinyJS_MathFunctions.cpp)

add_library(tinyjs_static STATIC ${headers} ${sources})
add_library(tinyjs_shared SHARED ${headers} ${sources})
set(libs tinyjs_static tinyjs_shared)

foreach (lib IN LISTS libs)
  set_target_properties(${lib} PROPERTIES OUTPUT_NAME tinyjs)
  target_compile_options(${lib} PRIVATE -Wall)
endforeach ()

install(
  TARGETS ${libs}
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")

install(
  FILES ${headers}
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/TinyJS")

# Add tests and examples
foreach (exe run_tests Script)
  add_executable(${exe} ${exe}.cpp)
  target_link_libraries(${exe} tinyjs_shared)
endforeach ()

