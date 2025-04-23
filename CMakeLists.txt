cmake_minimum_required(VERSION 3.15)
project(GKlib LANGUAGES C VERSION 5.0.0)

# -----------------------------------------------------------------------------
# Options
# -----------------------------------------------------------------------------
# Use standard BUILD_SHARED_LIBS for shared vs static
option(BUILD_SHARED_LIBS "Build shared libraries (.dll/.so) instead of static ones (.lib/.a)" OFF)

# GNUInstallDirs for standard installation directories
include(GNUInstallDirs)

# -----------------------------------------------------------------------------
# Gather sources and headers from GKlibSystem
# -----------------------------------------------------------------------------
include(GKlibSystem.cmake)  # sets GKlib_sources and GKlib_includes

# -----------------------------------------------------------------------------
# Define the GKlib library
# -----------------------------------------------------------------------------
file(GLOB GKlib_sources
  RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
  "${CMAKE_CURRENT_SOURCE_DIR}/*.c"
)
add_library(GKlib
  ${GKlib_sources}
)

target_include_directories(GKlib
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/GKlib>
)

# Windows-specific sources
if(MSVC)
  file(GLOB WIN32_SRC "${CMAKE_CURRENT_SOURCE_DIR}/win32/*.c")
  target_sources(GKlib PRIVATE ${WIN32_SRC})
  target_include_directories(GKlib PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/win32)
endif()

# Link math library on Unix
if(UNIX AND NOT APPLE)
  target_link_libraries(GKlib PUBLIC m)
endif()

# -----------------------------------------------------------------------------
# Testing (optional)
# -----------------------------------------------------------------------------
option(ENABLE_TESTING "Enable building of tests" ON)
if(ENABLE_TESTING)
  enable_testing()
  add_subdirectory(test)
endif()

# -----------------------------------------------------------------------------
# Installation
# -----------------------------------------------------------------------------
# Install library target
install(TARGETS GKlib
  EXPORT GKlibTargets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

# Install public headers
install(FILES ${GKlib_includes}
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/GKlib
)

# Export targets for find_package
install(EXPORT GKlibTargets
  NAMESPACE GKlib::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/GKlib
)

# Config file for find_package
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/GKlibConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/GKlibConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/GKlibConfig.cmake"
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/GKlib
)

# Install config files
install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/GKlibConfig.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/GKlibConfigVersion.cmake"
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/GKlib
)

