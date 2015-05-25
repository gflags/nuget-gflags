cmake_minimum_required (VERSION 3.0 FATAL_ERROR)

set (CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}/install" CACHE PATH "Path where to install binaries")

project (nuget-gflags)

include (ExternalProject)

set (GFLAGS_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../gflags" CACHE PATH "Path to gflags source code")

if(NOT EXISTS "${GFLAGS_SOURCE_DIR}")
    message(FATAL_ERROR "Invalid path to gflags sources")
endif()

set (CMAKE_GENERATORS 
    "Visual Studio 10 2010 Win64" "Visual Studio 10 2010" "Visual Studio 11 2012 Win64" "Visual Studio 11 2012" "Visual Studio 12 2013 Win64" "Visual Studio 12 2013"
    CACHE STRING 
    "List of generators to use"
    )

configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/gflags.autopkg ${CMAKE_CURRENT_BINARY_DIR}/gflags.autopkg @ONLY)
configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/gflags-config.cmake ${CMAKE_CURRENT_BINARY_DIR}/gflags-config.cmake @ONLY)
configure_file( ${CMAKE_CURRENT_SOURCE_DIR}/TargetCopySharedLibs.cmake ${CMAKE_CURRENT_BINARY_DIR}/TargetCopySharedLibs.cmake @ONLY)

foreach (_generator ${CMAKE_GENERATORS})

    string (REPLACE " " "" _generator_id ${_generator})
    # determine if we are 64 bit or 32 bit
    string (FIND ${_generator} "Win64" _arch_specifier)
    if(${_arch_specifier} LESS 0)
        set (_arch_specifier "Win32")
    else()
        set (_arch_specifier "x64")
    endif()
    
    # find platform toolset
    string(REGEX REPLACE "Visual Studio ([1][0-3]) .*" "v\\10" _platform_specifier ${_generator})    
    
    set (GFLAGS_CMAKE_ARGS 
         -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}/${_arch_specifier}/${_platform_specifier}/static # set the install prefix per generator
         -DCMAKE_DEBUG_POSTFIX=d # add a debug postfix to the debug libs so we can install them with the release ones
         -DBUILD_SHARED_LIBS=OFF # build static libs  
         )

    ExternalProject_Add ( gflags_static_${_arch_specifier}_${_platform_specifier}
                          SOURCE_DIR ${GFLAGS_SOURCE_DIR}
                          CMAKE_GENERATOR ${_generator}
                          CMAKE_ARGS ${GFLAGS_CMAKE_ARGS}
                        )

    set (GFLAGS_CMAKE_ARGS 
         -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}/${_arch_specifier}/${_platform_specifier}/dynamic # set the install prefix per generator
         -DCMAKE_DEBUG_POSTFIX=d # add a debug postfix to the debug libs so we can install them with the release ones
         -DBUILD_SHARED_LIBS=ON # build shared libs     
         )

    ExternalProject_Add ( gflags_shared_${_arch_specifier}_${_platform_specifier}
                          SOURCE_DIR ${GFLAGS_SOURCE_DIR}
                          CMAKE_GENERATOR ${_generator}
                          CMAKE_ARGS ${GFLAGS_CMAKE_ARGS}
                        )

endforeach()

