cmake_minimum_required(VERSION 3.24)
project(libmysofa C CXX)

include(CheckCCompilerFlag)
include(GenerateExportHeader)
include(GNUInstallDirs)

option(BUILD_TESTS "Build test programs" ON)
option(BUILD_SHARED_LIBS "Build shared library" ON)
option(BUILD_STATIC_LIBS "Build static library" ON)

set(CPACK_PACKAGE_DESCRIPTION "SOFA file reader for better HRTFs")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY
    "This library reads AES SOFA files and normalizes them")
set(CPACK_PACKAGE_VENDOR "Symonics GmbH")
set(CPACK_PACKAGE_CONTACT "umlaeute@debian.org")
set(CPACK_GENERATOR "DEB")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "IOhannes m zmölnig")
set(CPACK_PACKAGE_VERSION_MAJOR "1")
set(CPACK_PACKAGE_VERSION_MINOR "3")
set(CPACK_PACKAGE_VERSION_PATCH "2")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "zlib1g")
set(PKG_CONFIG_PRIVATELIBS "")

set(PROJECT_VERSION
    "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}")

check_c_compiler_flag(-Wall C_HAS_WALL)
if(C_HAS_WALL)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall")
endif(C_HAS_WALL)

set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -DDEBUG -DVDEBUG")

if (NOT MSVC AND NOT APPLE)
if(${ADDRESS_SANITIZE} STREQUAL "ON")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fsanitize=address -O1 -fno-omit-frame-pointer -g")
endif()
endif()

install(FILES share/default.sofa DESTINATION ${CMAKE_INSTALL_DATADIR}/libmysofa)
install(FILES share/MIT_KEMAR_normal_pinna.sofa
        DESTINATION ${CMAKE_INSTALL_DATADIR}/libmysofa)

if(BUILD_TESTS)

  enable_testing()

  add_test(Mesh2HRTF src/mysofa2json -c
           ${PROJECT_SOURCE_DIR}/tests/Mesh2HRTF.sofa)
endif(BUILD_TESTS)

add_subdirectory(src)

include(CPack)

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/libmysofa.pc.cmake"
               "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc" @ONLY)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig")

# Install exported targets as config files for use from install tree
install(EXPORT mysofa_exports
  NAMESPACE mysofa::
  FILE mysofaTargets.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/mysofa
)

# Create a target export file for use from the build tree.
export(EXPORT mysofa_exports
  NAMESPACE mysofa::
  FILE ${CMAKE_CURRENT_BINARY_DIR}/mysofaTargets.cmake
)

include(CMakePackageConfigHelpers)

# Configure package config file
configure_package_config_file(mysofaConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/mysofaConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/mysofa
)

# Create package version file
# Assuming major version bump = breaking API changes
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/mysofaConfigVersion.cmake
  VERSION ${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}
  COMPATIBILITY SameMajorVersion)

# Install package config file for use from install tree
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/mysofaConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/mysofaConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/mysofa
)
