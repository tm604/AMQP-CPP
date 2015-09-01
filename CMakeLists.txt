cmake_minimum_required(VERSION 2.8)

project(amqp-cpp)
set (VERSION_MAJOR 2)
set (VERSION_MINOR 2)
set (VERSION_PATCH 0)
set (PROJECT_VERSION "${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
set (PROJECT_VERSION_REVISION 2)

# ensure c++11 on all compilers
include(set_cxx_norm.cmake)
set_cxx_norm (${CXX_NORM_CXX11})

macro (add_sources)
    file (RELATIVE_PATH _relPath "${PROJECT_SOURCE_DIR}" "${CMAKE_CURRENT_SOURCE_DIR}")
    foreach (_src ${ARGN})
        if (_relPath)
            list (APPEND SRCS "${_relPath}/${_src}")
        else()
            list (APPEND SRCS "${_src}")
        endif()
    endforeach()
    if (_relPath)
        # propagate SRCS to parent directory
        set (SRCS ${SRCS} PARENT_SCOPE)
    endif()
endmacro()

add_subdirectory(src)
add_subdirectory(include)

add_library(amqp-cpp STATIC ${SRCS})
install(TARGETS amqp-cpp
        ARCHIVE DESTINATION lib
)

option(BUILD_TUTORIALS "build rabbitmq tutorials" OFF)
if(BUILD_TUTORIALS)
    add_subdirectory(examples/rabbitmq_tutorials)
endif()

set(AMQP-CPP_INCLUDE_PATH ${CMAKE_CURRENT_SOURCE_DIR})
set(AMQP-CPP_INCLUDE_PATH ${CMAKE_CURRENT_SOURCE_DIR} PARENT_SCOPE)

install(
	DIRECTORY include/
	DESTINATION "include/amqpcpp/"
)
install(
	FILES amqpcpp.h
	DESTINATION "include"
)

include (CTest)

include (InstallRequiredSystemLibraries)
set (CPACK_PACKAGE_NAME "libamqpcpp")
set (CPACK_PACKAGE_VENDOR "")
set (CPACK_PACKAGE_VERSION_MAJOR "${VERSION_MAJOR}")
set (CPACK_PACKAGE_VERSION_MINOR "${VERSION_MINOR}")
set (CPACK_PACKAGE_VERSION_PATCH "${VERSION_PATCH}")
set (CPACK_PACKAGE_CONTACT "Tom Molesworth <tom@perlsite.co.uk>")
set (CPACK_PACKAGE_DESCRIPTION_SUMMARY "Copernica's C++ abstract implementation for AMQP 0.9.1")
set (CPACK_PACKAGE_DESCRIPTION_FILE "${CMAKE_CURRENT_SOURCE_DIR}/README.debian")
set (CPACK_GENERATOR "TGZ;DEB")
string(TOLOWER "${CPACK_PACKAGE_NAME}" CPACK_PACKAGE_NAME_LOWERCASE)
find_program(DPKG_PROGRAM dpkg)
if(DPKG_PROGRAM)
	execute_process(
		COMMAND ${DPKG_PROGRAM} --print-architecture
		OUTPUT_VARIABLE CPACK_DEBIAN_PACKAGE_ARCHITECTURE
		OUTPUT_STRIP_TRAILING_WHITESPACE
		)
	set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME_LOWERCASE}_${PROJECT_VERSION}-${PROJECT_VERSION_REVISION}gcc5_${CPACK_DEBIAN_PACKAGE_ARCHITECTURE}")   
else(DPKG_PROGRAM)
	set(CPACK_PACKAGE_FILE_NAME "${CPACK_PACKAGE_NAME_LOWERCASE}_${PROJECT_VERSION}-${PROJECT_VERSION_REVISION}gcc5_${CMAKE_SYSTEM_NAME}")
endif(DPKG_PROGRAM)
# set (CPACK_DEBIAN_PACKAGE_SHLIBDEPS "ON")
include (CPack)
