
configure_file(config.h.in config.h)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

include(FindZLIB)
#HAve deleted libm. If any platform requires it look here
# https://stackoverflow.com/questions/32816646/can-cmake-detect-if-i-need-to-link-to-libm-when-using-pow-in-c

set(libsrc
    hrtf/reader.c
    hdf/superblock.c
    hdf/dataobject.c
    hdf/btree.c
    hdf/fractalhead.c
    hdf/gunzip.c
    hdf/gcol.c
    hrtf/check.c
    hrtf/spherical.c
    hrtf/lookup.c
    hrtf/tools.c
    hrtf/kdtree.c
    hrtf/neighbors.c
    hrtf/interpolate.c
    hrtf/resample.c
    hrtf/loudness.c
    hrtf/minphase.c
    hrtf/easy.c
    hrtf/cache.c
    resampler/speex_resampler.c)

set(public-headers
    hrtf/mysofa.h
)

if(BUILD_STATIC_LIBS)
add_library(mysofa-static STATIC ${libsrc})
target_include_directories(mysofa-static
  PUBLIC
  "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/hrtf>"
  $<INSTALL_INTERFACE:include/>
  PRIVATE 
  ${ZLIB_INCLUDE_DIR}
)
# Have deleted libm. If any platform requires it look here
# https://stackoverflow.com/questions/32816646/can-cmake-detect-if-i-need-to-link-to-libm-when-using-pow-in-c

target_link_libraries(mysofa-static PRIVATE ${ZLIB_LIBRARIES})
set_target_properties(
  mysofa-static
  PROPERTIES OUTPUT_NAME mysofa CLEAN_DIRECT_OUTPUT 1 POSITION_INDEPENDENT_CODE
                                                      ${BUILD_SHARED_LIBS})
set_target_properties( mysofa-static PROPERTIES PUBLIC_HEADER "${public-headers}" )
install(TARGETS mysofa-static
  EXPORT mysofa_exports
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )

endif()

if(BUILD_SHARED_LIBS)
  add_library(mysofa-shared SHARED ${libsrc})
  target_include_directories(mysofa-shared
    PUBLIC
    "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/hrtf>"
    $<INSTALL_INTERFACE:include/>
    PRIVATE
    ${ZLIB_INCLUDE_DIR}
  )
#  Have deleted libm. If any platform requires it look here
# https://stackoverflow.com/questions/32816646/can-cmake-detect-if-i-need-to-link-to-libm-when-using-pow-in-c

  target_link_libraries(mysofa-shared PRIVATE ${ZLIB_LIBRARIES})
  set_target_properties(mysofa-shared
                        PROPERTIES OUTPUT_NAME mysofa CLEAN_DIRECT_OUTPUT 1)
  # Rename the Windows import library (stub) of the DLL to prevent a name clash
  # with the mysofa-static library.
  set_target_properties(mysofa-shared
                        PROPERTIES ARCHIVE_OUTPUT_NAME mysofa_shared)

  set_property(
    TARGET mysofa-shared
    PROPERTY
      VERSION
      "${CPACK_PACKAGE_VERSION_MAJOR}.${CPACK_PACKAGE_VERSION_MINOR}.${CPACK_PACKAGE_VERSION_PATCH}"
  )
  set_property(TARGET mysofa-shared PROPERTY SOVERSION
                                             ${CPACK_PACKAGE_VERSION_MAJOR})
  set_property(TARGET mysofa-shared PROPERTY C_VISIBILITY_PRESET hidden)
  generate_export_header(mysofa-shared
    BASE_NAME MYSOFA
    EXPORT_FILE_NAME mysofa_export.h
    EXPORT_MACRO_NAME MYSOFA_EXPORT
    STATIC_DEFINE MYSOFA_STATIC
  )

  install(
    TARGETS mysofa-shared
    EXPORT mysofa_exports
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  )
else()
  generate_export_header(mysofa-static BASE_NAME mysofa EXPORT_FILE_NAME
                         mysofa_export.h)
endif()

if(BUILD_TESTS)
  add_executable(mysofa2json tests/sofa2json.c tests/json.c)
  if(BUILD_STATIC_LIBS)
    target_link_libraries(mysofa2json mysofa-static)
  else()
    target_link_libraries(mysofa2json mysofa-shared)
  endif()


  install(
    TARGETS mysofa2json
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
endif(BUILD_TESTS)

set(PKG_CONFIG_PRIVATELIBS
    "${PKG_CONFIG_PRIVATELIBS}"
    PARENT_SCOPE)
