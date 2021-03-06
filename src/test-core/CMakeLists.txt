SET(test_core_SOURCES
  test-stuff.c
  unittest-support.c
)

SET(test_core_noinst_HEADERS
  test-stuff.h
  unittest-support.h
)

INCLUDE_DIRECTORIES(
  ${CMAKE_BINARY_DIR}/src
  ${CMAKE_SOURCE_DIR}/src
  ${CMAKE_SOURCE_DIR}/src/libqof/qof
  ${CMAKE_SOURCE_DIR}/src/test-core

  ${GLIB2_INCLUDE_DIRS}
  ${GUILE_INCLUDE_DIRS}
)

ADD_LIBRARY(test-core STATIC ${test_core_SOURCES} ${test_core_noinst_HEADERS})
TARGET_LINK_LIBRARIES(test-core gncmod-engine gnc-qof ${GLIB2_LDFLAGS})
IF (UNIX)
  TARGET_COMPILE_OPTIONS(test-core PRIVATE -fPIC)
ENDIF()

SET (SWIG_UNITTEST_SUPPORT_GUILE_C ${CMAKE_CURRENT_BINARY_DIR}/swig-unittest-support-guile.c)
GNC_ADD_SWIG_COMMAND (${SWIG_UNITTEST_SUPPORT_GUILE_C}
    ${CMAKE_CURRENT_SOURCE_DIR}/unittest-support.i ${test_core_HEADERS})


SET(test_core_guile_SOURCES ${SWIG_UNITTEST_SUPPORT_GUILE_C})

ADD_LIBRARY(test-core-guile ${test_core_guile_SOURCES})
TARGET_LINK_LIBRARIES(test-core-guile test-core ${GUILE_LDFLAGS} ${GLIB2_LDFLAGS})

SET(test_core_SCHEME unittest-support.scm)

SET(GUILE_OUTPUT_DIR   gnucash)
SET(GUILE_MODULES      "")
SET(GUILE_LOAD_DIRS    "")
SET(GUILE_LIBRARY_DIRS src/test-core src/libqof)
SET(GUILE_DEPENDS      test-core-guile)


GNC_ADD_SCHEME_TARGETS(scm-test-core
  ${test_core_SCHEME}
  ${GUILE_OUTPUT_DIR}
  "${GUILE_MODULES}"
  "${GUILE_LOAD_DIRS}"
  "${GUILE_LIBRARY_DIRS}"
  "${GUILE_DEPENDS}"
  FALSE
  )

IF(GTEST_FOUND)
  IF(NOT GTEST_SHARED_LIB)
    SET (lib_gtest_SOURCES ${GTEST_SRC_DIR}/src/gtest-all.cc)
    ADD_LIBRARY(gtest STATIC  ${lib_gtest_SOURCES})
    TARGET_INCLUDE_DIRECTORIES(gtest PUBLIC ${GTEST_INCLUDE_DIR} ${GTEST_SRC_DIR})
  ENDIF()
  IF(GMOCK_FOUND)
    SET (lib_gmock_SOURCES ${GMOCK_SRC_DIR}/src/gmock-all.cc)
    ADD_LIBRARY(gmock STATIC  ${lib_gmock_SOURCES})
    TARGET_INCLUDE_DIRECTORIES(gmock PUBLIC
      ${GTEST_INCLUDE_DIR} ${GTEST_SRC_DIR}
      ${GMOCK_INCLUDE_DIR} ${GMOCK_SRC_DIR})
  ENDIF()
ENDIF()
INSTALL(FILES unittest-support.h DESTINATION libexec/gnucash/src/libqof/qof/test)
