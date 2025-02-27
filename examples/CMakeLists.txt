cmake_minimum_required(VERSION 3.11)

project(examples)

option(LINK_AGAINST_STATIC_LIBNFIQ2 "Link against static (vs shared) libnfiq2" YES)

# TODO: Add find_package for NFIQ2
set(NFIQ2_INSTALL_DIR /usr/local)
include_directories(${NFIQ2_INSTALL_DIR}/include)
################################################################################
set(REQUIRED_LIBS "")

if (LINK_AGAINST_STATIC_LIBNFIQ2)
	find_library(LIBNFIQ2_STATIC
		${CMAKE_STATIC_LIBRARY_PREFIX}nfiq2${CMAKE_STATIC_LIBRARY_SUFFIX}
		PATHS ${NFIQ2_INSTALL_DIR}/lib/nfiq2
		REQUIRED)
	find_library(LIBFMR_STATIC
		${CMAKE_STATIC_LIBRARY_PREFIX}fmr${CMAKE_STATIC_LIBRARY_SUFFIX}
		PATHS ${NFIQ2_INSTALL_DIR}/lib/nfiq2
		REQUIRED)
	find_library(LIBBIOMDI_STATIC
		${CMAKE_STATIC_LIBRARY_PREFIX}biomdi${CMAKE_STATIC_LIBRARY_SUFFIX}
		PATHS ${NFIQ2_INSTALL_DIR}/lib/nfiq2
		REQUIRED)
	find_library(LIBFRFXLL_STATIC
		${CMAKE_STATIC_LIBRARY_PREFIX}FRFXLL_static${CMAKE_STATIC_LIBRARY_SUFFIX}
		PATHS ${NFIQ2_INSTALL_DIR}/lib/nfiq2
		REQUIRED)

	find_package(OpenCV REQUIRED HINTS ${NFIQ2_INSTALL_DIR}/lib)
	set(OpenCV_SHARED ON)

	list(APPEND REQUIRED_LIBS
		${LIBNFIQ2_STATIC}
		${LIBFMR_STATIC}
		${LIBFRFXLL_STATIC}
		${LIBBIOMDI_STATIC}
		${OpenCV_LIBS})
else()
	find_library(LIBNFIQ2_SHARED
		nfiq2
		PATHS ${NFIQ2_INSTALL_DIR}/lib/nfiq2
		REQUIRED)

	list(APPEND REQUIRED_LIBS
		${LIBNFIQ2_SHARED})
endif()

set(ALL_TARGETS "")
################################################################################

#
# Tool that prints the names of all NFIQ 2 features.
#

add_executable(print_feature_names print_feature_names.cpp)
target_link_libraries(print_feature_names ${REQUIRED_LIBS})
set_target_properties(print_feature_names PROPERTIES
	MACOSX_RPATH YES
	CXX_STANDARD 11
	CXX_STANDARD_REQUIRED YES
	CXX_EXTENSIONS NO)
list(APPEND ALL_TARGETS print_feature_names)

################################################################################

# Enable warnings for all targets
if (CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
	set(WARNING_FLAGS -Wall -Wextra -Wconversion -Wsign-conversion -pedantic)
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
	set(WARNING_FLAGS /W4)
endif()
if (WARNING_FLAGS)
	set_target_properties(${ALL_TARGETS} PROPERTIES
		COMPILE_OPTIONS "${WARNING_FLAGS}")
endif()

################################################################################
