cmake_minimum_required(VERSION 3.13)
#cmake_minimum_required(VERSION 3.14)
project(vufuse2fs
    VERSION 0.2
    DESCRIPTION "fuse/vuos (reentrant) module for ext2/ext3/ext4"
    HOMEPAGE_URL "https://github.com/virtualsquare/vufuse2fs"
    LANGUAGES C)

include(GNUInstallDirs)
include(CheckIncludeFile)
include_directories(${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/include
		${CMAKE_CURRENT_BINARY_DIR})

find_package(PkgConfig)
pkg_check_modules(FUSE REQUIRED fuse3)
string(REGEX REPLACE "\\..*" "" FUSE_VERSION ${FUSE_VERSION})

include(CheckIncludeFile)
include(CheckSymbolExists)

set(LIBS_REQUIRED ext2fs uuid blkid com_err)
set(HEADERS_REQUIRED ext2fs/ext2fs.h et/com_err.h)

foreach(THISLIB IN LISTS LIBS_REQUIRED)
  find_library(LIB${THISLIB}_OK ${THISLIB})
  if(NOT LIB${THISLIB}_OK)
    message(FATAL_ERROR "library lib${THISLIB} not found")
  endif()
endforeach(THISLIB)

foreach(HEADER IN LISTS HEADERS_REQUIRED)
  check_include_file(${HEADER} ${HEADER}_OK)
  if(NOT ${HEADER}_OK)
    message(FATAL_ERROR "header file ${HEADER} not found")
  endif()
endforeach(HEADER)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -D_FORTIFY_SOURCE=2 -O2 -pedantic -Wall -Wextra")

add_definitions(-D_GNU_SOURCE -D_FILE_OFFSET_BITS=64 ${FUSE_CFLAGS} -DFUSE=${FUSE_VERSION})

add_executable(refuse2fs fuse2fs.c fake_ext2fs_run_ext3_journal.c)
target_link_libraries(refuse2fs ${FUSE_LIBRARIES} ext2fs uuid blkid com_err)
install(TARGETS refuse2fs
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

add_library(vufuse2fs SHARED fuse2fs.c fake_ext2fs_run_ext3_journal.c vufuse_reentrant.c)
target_link_libraries(vufuse2fs ${FUSE_LIBRARIES} ext2fs uuid blkid com_err)
set_target_properties(vufuse2fs PROPERTIES PREFIX "")
install(TARGETS vufuse2fs
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}/vu/modules)

add_subdirectory(man)

add_custom_target(uninstall
  "${CMAKE_COMMAND}" -P "${PROJECT_SOURCE_DIR}/Uninstall.cmake"
  )
