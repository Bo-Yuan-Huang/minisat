cmake_minimum_required(VERSION 3.1 FATAL_ERROR)

if(${CMAKE_VERSION} VERSION_LESS 3.16)
  cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
else()
  cmake_policy(VERSION 3.16)
endif()


# ---------------------------------------------------------------------------- #
# PROJECT
# name version language
# ---------------------------------------------------------------------------- #
project(minisat VERSION   2.2.0
                LANGUAGES CXX
)


# ---------------------------------------------------------------------------- #
# OPTION
# build configuration
# ---------------------------------------------------------------------------- #
option(BUILD_SHARED_LIBS "Build shared libraries."                           ON)


# ---------------------------------------------------------------------------- #
# VARARIABLE
# alias and configurations
# ---------------------------------------------------------------------------- #
set(MINISAT_TARGET_NAME                ${PROJECT_NAME})
set(MINISAT_TARGETS_EXPORT_NAME        "${PROJECT_NAME}Targets")
set(MINISAT_INCLUDE_BUILD_DIR          "${PROJECT_SOURCE_DIR}")
set(MINISAT_INCLUDE_INSTALL_DIR        "include")
set(MINISAT_CMAKE_CONFIG_TEMPLATE      "cmake/config.cmake.in")
set(MINISAT_CMAKE_CONFIG_DIR           "${CMAKE_CURRENT_BINARY_DIR}")
set(MINISAT_CMAKE_PROJECT_TARGETS_FILE "${PROJECT_NAME}Targets.cmake")

set(MINISAT_CONFIG_INSTALL_DIR         
  "lib/cmake/${PROJECT_NAME}" CACHE INTERNAL "")

set(MINISAT_CMAKE_VERSION_CONFIG_FILE
  "${MINISAT_CMAKE_CONFIG_DIR}/${PROJECT_NAME}ConfigVersion.cmake")

set(MINISAT_CMAKE_PROJECT_CONFIG_FILE
  "${MINISAT_CMAKE_CONFIG_DIR}/${PROJECT_NAME}Config.cmake")

if(${BUILD_SHARED_LIBS})
  set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
  set(CMAKE_MACOSX_RPATH TRUE)
endif()


# ---------------------------------------------------------------------------- #
# Dependencies
# ---------------------------------------------------------------------------- #
##
## Make sure that custom modules can be found.
##
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

##
## ZLIB
##
find_package(ZLIB)


# ---------------------------------------------------------------------------- #
# TARGET
# library 
# ---------------------------------------------------------------------------- #
set(MINISAT_LIB_NAME ${MINISAT_TARGET_NAME})

add_library(${MINISAT_LIB_NAME} 
  minisat/utils/Options.cc
  minisat/utils/System.cc
  minisat/core/Solver.cc
  minisat/simp/SimpSolver.cc
)

add_library(${PROJECT_NAME}::${MINISAT_LIB_NAME} ALIAS ${MINISAT_LIB_NAME})

##
## compile features/options
##
if(${CMAKE_COMPILER_IS_GNUCXX})

  target_compile_options(${MINISAT_LIB_NAME} PRIVATE "-D__STDC_FORMAT_MACROS")
  target_compile_options(${MINISAT_LIB_NAME} PRIVATE "-D__STDC_LIMIT_MACROS")

  if(${CMAKE_PROJECT_NAME} MATCHES ${PROJECT_NAME})
    if(${CMAKE_BUILD_TYPE} MATCHES Debug)
      target_compile_options(${MINISAT_LIB_NAME} PRIVATE "-Wall")
    endif()
  endif()

endif()

##
## include paths
##
target_include_directories(${MINISAT_LIB_NAME} 
  PUBLIC 
  $<INSTALL_INTERFACE:${MINISAT_INCLUDE_INSTALL_DIR}>
  $<BUILD_INTERFACE:${MINISAT_INCLUDE_BUILD_DIR}>
)

##
## link dependencies
##
target_link_libraries(${MINISAT_LIB_NAME} PUBLIC ZLIB::ZLIB)

##
## install
##
install(
  TARGETS  ${MINISAT_LIB_NAME}
  EXPORT   ${MINISAT_TARGETS_EXPORT_NAME}
  LIBRARY  DESTINATION lib
  ARCHIVE  DESTINATION lib
  RUNTIME  DESTINATION bin
  INCLUDES DESTINATION ${MINISAT_INCLUDE_INSTALL_DIR}
)

install(
  DIRECTORY ${MINISAT_INCLUDE_BUILD_DIR}/minisat
  DESTINATION ${MINISAT_INCLUDE_INSTALL_DIR}
)


# ---------------------------------------------------------------------------- #
# TARGET
# executable
# ---------------------------------------------------------------------------- #
add_executable(minisat_core minisat/core/Main.cc)
add_executable(minisat_simp minisat/simp/Main.cc)

target_link_libraries(minisat_core ${MINISAT_LIB_NAME})
target_link_libraries(minisat_simp ${MINISAT_LIB_NAME})


# ---------------------------------------------------------------------------- #
# INSTALL
# project 
# ---------------------------------------------------------------------------- #
##
## forward version for find_package
##
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${MINISAT_CMAKE_VERSION_CONFIG_FILE} 
  COMPATIBILITY SameMajorVersion # AnyNewVersion
)

##
## cmake config
##
configure_file(
  ${MINISAT_CMAKE_CONFIG_TEMPLATE}
  ${MINISAT_CMAKE_PROJECT_CONFIG_FILE}
  @ONLY
)

install(
  FILES ${MINISAT_CMAKE_PROJECT_CONFIG_FILE} ${MINISAT_CMAKE_VERSION_CONFIG_FILE}
  DESTINATION ${MINISAT_CONFIG_INSTALL_DIR}
)

##
## project export
##
install(
  EXPORT ${MINISAT_TARGETS_EXPORT_NAME}
  DESTINATION ${MINISAT_CONFIG_INSTALL_DIR}
  NAMESPACE ${PROJECT_NAME}::
  FILE ${MINISAT_CMAKE_PROJECT_TARGETS_FILE}
)

export(
  TARGETS ${MINISAT_LIB_NAME}
  NAMESPACE ${PROJECT_NAME}::
  FILE ${MINISAT_CMAKE_PROJECT_TARGETS_FILE}
)


# ---------------------------------------------------------------------------- #
# UNINSTALL
# uninstall files listed in install_manifest.txt
# ---------------------------------------------------------------------------- #
if(NOT TARGET uninstall)
  configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake/cmake_uninstall.cmake"
    IMMEDIATE @ONLY
  )

  add_custom_target(uninstall
    COMMAND 
      ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake/cmake_uninstall.cmake
  )

endif()

