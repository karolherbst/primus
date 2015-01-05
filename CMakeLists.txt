project(primus)
cmake_minimum_required(VERSION 2.8)

include(GNUInstallDirs)

find_package(Threads REQUIRED)
find_package(X11 REQUIRED)

macro(add_linker_flags)
  foreach(flag ${ARGN})
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${flag}")
    set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} ${flag}")
  endforeach()
endmacro()

set(CMAKE_CXX_FLAGS "-Werror=missing-declarations -Werror=attributes -fvisibility=hidden")
add_linker_flags("-Wl,--as-needed -Wl,-z,defs -Wl,-Bsymbolic")

set(bitness 32)
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(bitness 64)
endif()

set(BUMBLEBEE_SOCKET "/var/run/bumblebee.socket" CACHE STRING "Path to bumblebee socket")
set(PRIMUS_SYNC "0" CACHE STRING "Default value for PRIMUS_SYNC")
set(PRIMUS_VERBOSE "1" CACHE STRING "Default value for PRIMUS_VERBOSE")
set(PRIMUS_UPLOAD "0" CACHE STRING "Default value for PRIMUS_UPLOAD")
set(PRIMUS_SLEEP "90" CACHE STRING "Default value for PRIMUS_SLEEP")
set(PRIMUS_DISPLAY ":8" CACHE STRING "Default value for PRIMUS_DISPLAY")
set(PRIMUS_LOAD_GLOBAL "libglapi.so.0" CACHE STRING "Default value of PRIMUS_LOAD_GLOBAL")
set(PRIMUS_libGLa "/usr/lib${bitness}/nvidia/libGL.so.1" CACHE STRING "Path to nvidia libGL.so")
set(PRIMUS_libGLd "/usr/lib${bitness}/libGL.so.1" CACHE STRING "Path to intel libGL.so")

add_definitions(
  -DBUMBLEBEE_SOCKET="${BUMBLEBEE_SOCKET}"
  -DPRIMUS_SYNC="${PRIMUS_SYNC}"
  -DPRIMUS_VERBOSE="${PRIMUS_VERBOSE}"
  -DPRIMUS_UPLOAD="${PRIMUS_UPLOAD}"
  -DPRIMUS_SLEEP="${PRIMUS_SLEEP}"
  -DPRIMUS_DISPLAY="${PRIMUS_DISPLAY}"
  -DPRIMUS_LOAD_GLOBAL="${PRIMUS_LOAD_GLOBAL}"
  -DPRIMUS_libGLa="${PRIMUS_libGLa}"
  -DPRIMUS_libGLd="${PRIMUS_libGLd}"
)

add_library(GL SHARED ${CMAKE_CURRENT_SOURCE_DIR}/libglfork.cpp)
set_target_properties(GL PROPERTIES
  SOVERSION 1
  VERSION 1.2.0
)

target_link_libraries(GL
  ${X11_X11_LIB}
  ${CMAKE_THREAD_LIBS_INIT}
  ${CMAKE_DL_LIBS}
)

install(TARGETS GL
  LIBRARY DESTINATION ${CMAKE_INSTALL_FULL_LIBDIR}/primus
)

install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/primusrun
  DESTINATION ${CMAKE_INSTALL_FULL_BINDIR}
  PERMISSIONS OWNER_EXECUTE OWNER_WRITE OWNER_READ
              GROUP_EXECUTE             GROUP_READ
              WORLD_EXECUTE             WORLD_READ
)
