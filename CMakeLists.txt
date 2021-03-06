##################################################################################
# "THE ANY BEVERAGE-WARE LICENSE" (Revision 42 - based on beer-ware
# license):
# <dev@layer128.net> wrote this file. As long as you retain this notice
# you can do whatever you want with this stuff. If we meet some day, and
# you think this stuff is worth it, you can buy me a be(ve)er(age) in
# return. (I don't like beer much.)
#
# Matthias Kleemann
##################################################################################

##################################################################################
# Modified by Jacob Brown (jb21g14) on 14/05/2018
# For use by Computer System 2 Students at the University of Southampton
# All settings are based on an at90usb1286 chip, using dfu-programmer as the upload tool
#
# N.B.: The CMake proejct needs to be reloaded every time a new source file is created
#
# Setup Instructions (CLion):
# create a new CLion project
# Set the cmake_minimum_required on line 37 to the version of cmake you are using e.g. "cmake_minimum_required(VERSION 3.0)"
# Set the name on line 59 of this file to the CLion project name you set above e.g. "project(Week4_task C)"
# copy this and the generic-gcc-avr.cmake into the root folder directory of the CLion project overwriting any conflicts
# Reload the CMake project in CLion
#
# Setup Instructions (general):
# Set the cmake_minimum_required on line 37 to the version of cmake you are using e.g. "cmake_minimum_required(VERSION 3.0)"
# Set the name on line 51 of this file to anything you like e.g. "project(Week4_task C)"
#
# Any issues you can email me at jb21g14@soton.ac.uk or create an issue in the github project
##################################################################################

##################################################################################
# Sample CMakeLists.txt for a simple AVR project based on the toolchain
##################################################################################

cmake_minimum_required(VERSION CHANGE_ME)

### TOOLCHAIN SETUP AREA #################################################
# Set any variables used in the toolchain prior project() call. In that
# case they are already set and used.
##########################################################################

##################################################################################
# tools to be used for programming the AVR
##################################################################################
set(AVR_UPLOADTOOL dfu-programmer)
set(AVR_UPLOADTOOL_PORT usb)
#set(AVR_UPLOADTOOL_OPTIONS)
set(AVR_MCU at90usb1286)
set(F_CPU 8000000UL)
### END TOOLCHAIN SETUP AREA #############################################

include(generic-gcc-avr.cmake)

##########################################################################
# name your project
##########################################################################
project(CHANGE_ME C)


##################################################################################
# status messages
##################################################################################
message(STATUS "Current uploadtool is: ${AVR_UPLOADTOOL}")
message(STATUS "Current upload port is: ${AVR_UPLOADTOOL_PORT}")
message(STATUS "Current uploadtool options are: ${AVR_UPLOADTOOL_OPTIONS}")
message(STATUS "Current MCU is set to: ${AVR_MCU}")

##################################################################################
# set build type, if not already set at cmake command line
##################################################################################
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif(NOT CMAKE_BUILD_TYPE)

##################################################################################
# some cmake cross-compile necessities
##################################################################################
if(DEFINED ENV{AVR_FIND_ROOT_PATH})
    set(CMAKE_FIND_ROOT_PATH $ENV{AVR_FIND_ROOT_PATH})
else(DEFINED ENV{AVR_FIND_ROOT_PATH})
    if(EXISTS "/opt/local/avr")
        set(CMAKE_FIND_ROOT_PATH "/opt/local/avr")
    elseif(EXISTS "/usr/avr")
        set(CMAKE_FIND_ROOT_PATH "/usr/avr")
    elseif(EXISTS "/usr/lib/avr")
        set(CMAKE_FIND_ROOT_PATH "/usr/lib/avr")
    elseif(EXISTS "/usr/local/CrossPack-AVR")
        set(CMAKE_FIND_ROOT_PATH "/usr/local/CrossPack-AVR")
    else(EXISTS "/opt/local/avr")
        message(FATAL_ERROR "Please set AVR_FIND_ROOT_PATH in your environment.")
    endif(EXISTS "/opt/local/avr")
endif(DEFINED ENV{AVR_FIND_ROOT_PATH})
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
# not added automatically, since CMAKE_SYSTEM_NAME is "generic"
set(CMAKE_SYSTEM_INCLUDE_PATH "${CMAKE_FIND_ROOT_PATH}/include")
set(CMAKE_SYSTEM_LIBRARY_PATH "${CMAKE_FIND_ROOT_PATH}/lib")

##################################################################################
# status messages for generating
##################################################################################
message(STATUS "Set CMAKE_FIND_ROOT_PATH to ${CMAKE_FIND_ROOT_PATH}")
message(STATUS "Set CMAKE_SYSTEM_INCLUDE_PATH to ${CMAKE_SYSTEM_INCLUDE_PATH}")
message(STATUS "Set CMAKE_SYSTEM_LIBRARY_PATH to ${CMAKE_SYSTEM_LIBRARY_PATH}")

##################################################################################
# set compiler options for specific build types
##################################################################################

## Unecessary for our purposes

##################################################################################
# compiler options for all build types
##################################################################################
set(CSTANDARD "")
set(CDEBUG    "")
set(CWARN     "-Wall -Wextra -Wno-main -Wstrict-overflow=5 -Winline")
set(CTUNING   "-fstrict-overflow -fno-strict-aliasing")
set(COPT      "-Os")
set(CMCU      "-mmcu=${MCU}")
#set(CDEFS     "-DF_CPU=${F_CPU}") Have to set this using the line below otherwise we get errors about not defining F_CPU
add_definitions(-DF_CPU=${F_CPU})

set(CMAKE_C_CFLAGS "${CMCU} ${CDEBUG} ${CDEFS} ${COPT} ${CWARN} ${CSTANDARD} ${CTUNING}")



### File Discovery
# Files to be compiled - The following lines allow for the project structure to be managed using directories upon subdirectories
# See https://stackoverflow.com/questions/6921695/how-can-i-build-a-c-project-with-multiple-interdependent-subdirectories
file(GLOB_RECURSE SRC_FILES "src/*.c")
file(GLOB_RECURSE HEADER_FILES "src/*.h")

set (INCLUDE_DIRS "")
foreach (_headerFile ${HEADER_FILES})
    get_filename_component(_dir ${_headerFile} PATH)
    list (APPEND INCLUDE_DIRS ${_dir})
endforeach()
list(REMOVE_DUPLICATES INCLUDE_DIRS)

include_directories(${INCLUDE_DIRS})

##################################################################################
# add AVR executable
##################################################################################
add_avr_executable(${PROJECT_NAME} ${SRC_FILES})

