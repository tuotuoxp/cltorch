cmake_minimum_required(VERSION 2.8.12)

OPTION(DEV_RUN_COG "cltorch maintainers only, otherwise set to 'OFF'." OFF)

if("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
  set(ON_LINUX 1)
endif()
if("${CMAKE_SYSTEM_NAME}" STREQUAL "Windows")
  set(ON_WINDOWS 1)
endif()

# https://cmake.org/Wiki/CMake_RPATH_handling
SET(CMAKE_INSTALL_RPATH "${Torch_INSTALL_LUA_CPATH_SUBDIR}}/../..")
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
SET(CMAKE_MACOSX_RPATH TRUE)

#SET(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake" "${CMAKE_MODULE_PATH}")

SET(CMAKE_C_FLAGS "-std=c99 -Werror=implicit-function-declaration")
SET(CMAKE_CXX_FLAGS "-std=c++0x -Wall")

SET(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/src/cmake")

if(UNIX)
    execute_process(COMMAND git --version RESULT_VARIABLE git_test ERROR_QUIET)
    if(${git_test} EQUAL 0)
        if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/.git")
            message("using git setting USE_GIT 1")
            SET(USE_GIT 1)
        endif()
    endif()
endif()

if(UNIX)
  execute_process(COMMAND bash -c "${CMAKE_PREFIX_PATH}/bin/luarocks list distrocheck | grep distrocheck"
    RESULT_VARIABLE DISTROCHECK)
  if(${DISTROCHECK} EQUAL "1")
    message("CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH}")
    execute_process(COMMAND bash -c "${CMAKE_PREFIX_PATH}/bin/luarocks list")
    message(FATAL_ERROR "
    Please install cltorch from https://github.com/hughperkins/distro-cl")
  endif()
else()  # windows?
  execute_process(COMMAND "luarocks list distrocheck | find /i \"distrocheck\""
    RESULT_VARIABLE DISTROCHECK)
  if(${DISTROCHECK} EQUAL "1")
    message("CMAKE_PREFIX_PATH ${CMAKE_PREFIX_PATH}")
    execute_process(COMMAND "luarocks list")
    message(FATAL_ERROR "
    Please install cltorch from https://github.com/hughperkins/distro-cl")
  endif()
endif()

if(USE_GIT)
    message("using git")
    execute_process(COMMAND bash -c "echo $PWD"
      WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")
    execute_process(COMMAND git status
      WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")
        execute_process(
          COMMAND git submodule update --init --force --recursive
      WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
          )
endif()


INCLUDE("${CMAKE_MODULE_PATH}/build_clBLAS.cmake")
INCLUDE_DIRECTORIES(${clBLAS_INCLUDE_DIRS})
#LINK_DIRECTORIES(${CLBLAS_LIBRARY_DIR})

INCLUDE("${CMAKE_MODULE_PATH}/build_EasyCL.cmake")
INCLUDE_DIRECTORIES(${EasyCL_INCLUDE_DIRS})
#LINK_DIRECTORIES(${EASYCL_LIBRARY_DIR})

if(NOT TH_FOUND)
  find_package(TH REQUIRED)
endif()

INCLUDE_DIRECTORIES(${TH_INCLUDE_DIR})

SET(Torch_INSTALL_INCLUDE_SUBDIR "include")
SET(Torch_INSTALL_BIN_SUBDIR "bin")
SET(Torch_INSTALL_LIB_SUBDIR "lib")

ADD_SUBDIRECTORY(src)

