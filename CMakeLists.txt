# Project Name
project(Pcsx2)

# Debian-based distributions require at least 2.8.5 due to multiarch.
# Bumping up to 3.0 seems reasonable at this point, and will let us modernize
# things a bit.
#
# Setting it to a range tells it that it supports the features on the newer
# versions as well, avoiding setting policies.
cmake_minimum_required(VERSION 3.10...3.17)

# Variable to check that people use the good file
set(TOP_CMAKE_WAS_SOURCED TRUE)
set(CMAKE_CXX_VISIBILITY_PRESET hidden)
set(CMAKE_C_VISIBILITY_PRESET hidden)

# set module path
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)

# include some generic functions to ensure correctness of the env
include(Pcsx2Utils)

check_no_parenthesis_in_path()
detectOperatingSystem()
check_compiler_version("7.0" "7.0")

#-------------------------------------------------------------------------------
# Include specific module
# GNUInstallDirs must be done before BuildParameters
include(GNUInstallDirs)
# BuildParameters Must be done before SearchForStuff
include(BuildParameters)
include(SearchForStuff)

# Must be done after SearchForStuff
get_git_version_info()
write_svnrev_h()

if(NOT NO_TRANSLATION AND NOT PCSX2_CORE)
	# make the translation
	if(EXISTS "${CMAKE_SOURCE_DIR}/locales")
		add_subdirectory(locales)
	endif()
endif()

# make common
add_subdirectory(common)

# make pcsx2
add_subdirectory(pcsx2)

if (QT_BUILD)
	add_subdirectory(pcsx2-qt)
endif()

# tests
if(ACTUALLY_ENABLE_TESTS)
	set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
	add_subdirectory(3rdparty/gtest EXCLUDE_FROM_ALL)
	add_subdirectory(tests/ctest)
endif()

#-------------------------------------------------------------------------------

# Install some files to ease package creation
if(PACKAGE_MODE)
	INSTALL(DIRECTORY "${CMAKE_SOURCE_DIR}/bin/resources" DESTINATION "${CMAKE_INSTALL_DATADIR}/PCSX2")

	# set categories depending on system/distribution in pcsx2.desktop
	if(openSUSE)
		set(PCSX2_MENU_CATEGORIES "System;Emulator;")
	else()
		set(PCSX2_MENU_CATEGORIES "Game;Emulator;")
	endif()
	configure_file("${CMAKE_SOURCE_DIR}/linux_various/PCSX2.desktop.in" "${CMAKE_BINARY_DIR}/linux_various/PCSX2.desktop" @ONLY)

	INSTALL(FILES "${CMAKE_BINARY_DIR}/linux_various/PCSX2.desktop" DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/applications")
	INSTALL(FILES "${CMAKE_SOURCE_DIR}/linux_various/PCSX2.xpm"     DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/pixmaps")
	INSTALL(FILES "${CMAKE_SOURCE_DIR}/bin/docs/PCSX2_FAQ.pdf"      DESTINATION "${CMAKE_INSTALL_DOCDIR}")
	INSTALL(FILES "${CMAKE_SOURCE_DIR}/bin/docs/Configuration_Guide.pdf"   DESTINATION "${CMAKE_INSTALL_DOCDIR}")
	INSTALL(FILES "${CMAKE_SOURCE_DIR}/bin/docs/PCSX2.1"            DESTINATION "${CMAKE_INSTALL_MANDIR}/man1")
	if(NOT DISABLE_PCSX2_WRAPPER)
		INSTALL(FILES "${CMAKE_SOURCE_DIR}/linux_various/PCSX2-linux.sh"          DESTINATION "${CMAKE_INSTALL_BINDIR}")
	endif()

else()
	if(NOT DISABLE_PCSX2_WRAPPER)
		# special case to avoid having linux files in windows
		INSTALL(FILES "${CMAKE_SOURCE_DIR}/linux_various/PCSX2-linux.sh"          DESTINATION "${CMAKE_SOURCE_DIR}/bin")
	endif()
endif()
