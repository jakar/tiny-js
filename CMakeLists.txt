# Lowest CMake version tried. Lower versions might work.
cmake_minimum_required(VERSION 3.8)

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
  target_compile_features(${lib} PUBLIC cxx_std_14)
  target_compile_options(${lib} PRIVATE -Wall)
endforeach ()

install(
  TARGETS ${libs}
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")

install(
  FILES ${headers}
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

# Add tests and examples
add_executable(tinyjs_cli Script.cpp)
target_link_libraries(tinyjs_cli tinyjs_static)
set_target_properties(tinyjs_cli PROPERTIES OUTPUT_NAME tinyjs)
target_compile_features(tinyjs_cli PRIVATE cxx_std_14)
target_compile_options(tinyjs_cli PRIVATE -Wall)

add_executable(tinyjs_tests run_tests.cpp)
target_link_libraries(tinyjs_tests tinyjs_static)
set_target_properties(tinyjs_tests PROPERTIES OUTPUT_NAME tinyjs-tests)
target_compile_features(tinyjs_tests PRIVATE cxx_std_14)
target_compile_options(tinyjs_tests PRIVATE -Wall)

add_custom_command(
  TARGET tinyjs_tests POST_BUILD
  COMMAND "${CMAKE_COMMAND}" -E copy_directory "${CMAKE_SOURCE_DIR}/tests" "${CMAKE_CURRENT_BINARY_DIR}/tests")
