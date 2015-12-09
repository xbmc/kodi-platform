project(kodiplatform)

cmake_minimum_required(VERSION 2.6)
enable_language(CXX)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR})

find_package(kodi REQUIRED)
find_package(TinyXML REQUIRED)
find_package(Threads REQUIRED)
find_package(p8-platform REQUIRED)
include(UseMultiArch.cmake)
include(CheckAtomic.cmake)

set(kodiplatform_NAME kodiplatform)
set(kodiplatform_DESCRIPTION "Kodi add-on platform support library")
set(kodiplatform_VERSION_MAJOR ${APP_VERSION_MAJOR})
set(kodiplatform_VERSION_MINOR ${APP_VERSION_MINOR})
set(kodiplatform_VERSION_PATCH 0)

if(NOT WIN32)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
endif()

set(kodiplatform_INCLUDE_DIRS ${TINYXML_INCLUDE_DIR} "${CMAKE_INSTALL_PREFIX}/include/kodi")
IF(WIN32)
  LIST(APPEND kodiplatform_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include/kodi/windows")
ENDIF(WIN32)
set(kodiplatform_LIBRARIES ${CMAKE_THREAD_LIBS_INIT} ${TINYXML_LIBRARIES})

if(NOT ${CORE_SYSTEM_NAME} STREQUAL "")
  if(${CORE_SYSTEM_NAME} STREQUAL "osx" OR ${CORE_SYSTEM_NAME} STREQUAL "ios")
    list(APPEND kodiplatform_LIBRARIES "-framework CoreVideo -framework IOKit")
  endif()
endif()

set(SOURCES src/util/XMLUtils.cpp)

include_directories(${TINYXML_INCLUDE_DIR} ${KODI_INCLUDE_DIR} ${p8-platform_INCLUDE_DIRS})

add_library(kodiplatform ${SOURCES} ${PLAT_SOURCES})
target_link_libraries(kodiplatform ${kodiplatform_LIBRARIES} ${platform_LIBRARIES})
set_target_properties(kodiplatform PROPERTIES VERSION ${kodiplatform_VERSION_MAJOR}.${kodiplatform_VERSION_MINOR}.${kodiplatform_VERSION_PATCH}
                                              SOVERSION ${kodiplatform_VERSION_MAJOR}.0)

install(TARGETS kodiplatform DESTINATION ${CMAKE_INSTALL_LIBDIR})
install(FILES src/util/XMLUtils.h
        DESTINATION include/kodi/util)

IF(NOT WIN32)
  # Pkgconfig
  include(PkgConfigHandler.cmake)
  configure_pc_file(kodiplatform kodiplatform.pc.in
                                 kodiplatform.pc
                                 ${CMAKE_INSTALL_PREFIX}
                                 ${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}
                                 ${CMAKE_INSTALL_PREFIX}/include)

  install(FILES ${CMAKE_BINARY_DIR}/kodiplatform.pc
          DESTINATION ${CMAKE_INSTALL_LIBDIR_NOARCH}/pkgconfig)
ENDIF(NOT WIN32)

# config mode
configure_file (kodiplatform-config.cmake.in
                kodiplatform-config.cmake @ONLY)
install(FILES ${CMAKE_BINARY_DIR}/kodiplatform-config.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR_NOARCH}/kodiplatform)

# Quell warnings with in-tree builds
set(KODI_BUILD_DIR ${KODI_BUILD_DIR})
