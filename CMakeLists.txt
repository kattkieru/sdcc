cmake_minimum_required(VERSION 3.4)
project(gbdk3-sdcc)

include(ExternalProject)

set( BOOST_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../boost_1_64_0/ )
set( SDCC_CPP_FLAGS -I${BOOST_ROOT} )
set( SDCC_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/sdcc-3.6.1/sdcc )
set( SDCC_PREFIX ${CMAKE_CURRENT_SOURCE_DIR}/.. )

message( AUTHOR_WARNING "Current CMake Source Directory: ${CMAKE_CURRENT_SOURCE_DIR}." )
message( AUTHOR_WARNING "Boost Directory: ${BOOST_ROOT}." )

set( SDCC_CONFIGURE_COMMAND 
	configure
	--prefix=${CMAKE_CURRENT_SOURCE_DIR}/.. 
	--disable-mcs51-port 
	--disable-z80-port 
	--disable-z180-port 
	--disable-r2k-port 
	--disable-r3ka-port 
	--disable-tlcs90-port 
	--disable-ds390-port 
	--disable-ds400-port 
	--disable-pic14-port 
	--disable-pic16-port 
	--disable-hc08-port 
	--disable-s08-port 
	--disable-stm8-port 
	--disable-ucsim 
	--disable-device-lib
	--disable-non-free
)

## this is just to make the above readable
## lists of string are stored as item1;item2;item3 strings,
## so you need to replace the semi-colon with a space
string(REPLACE ";" " " SDCC_CONFIGURE_COMMAND_FLAT "${SDCC_CONFIGURE_COMMAND}")

## There is literally no good way to force an environment
## variable into a cmake build that calls an external
## project using configure, so I'm punting and using
## a shell script to bootstrap the build.
set( SDCC_BOOTSTRAP_SCRIPT "" )
set( SDCC_ENV_COMMAND "" )

if (WIN32)
	message( AUTHOR_WARNING "+ Building for Windows." )
	error( "Not yet supported." )

elseif( UNIX AND APPLE )
	set( SDCC_BOOTSTRAP_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/bootstrap.sh )
	set( SDCC_SHELL "/bin/bash" )
	set( SDCC_ENV_COMMAND "export CXXFLAGS=\"-I${BOOST_ROOT} -Wmicrosoft-anon-tag\"" )

	file( WRITE  ${SDCC_BOOTSTRAP_SCRIPT} "#!${SDCC_SHELL}\n" )
	file( APPEND ${SDCC_BOOTSTRAP_SCRIPT} "cd ${SDCC_SOURCE_DIR}\n" )
	file( APPEND ${SDCC_BOOTSTRAP_SCRIPT} "${SDCC_ENV_COMMAND}\n" )
	file( APPEND ${SDCC_BOOTSTRAP_SCRIPT} "./${SDCC_CONFIGURE_COMMAND_FLAT}\n" )

	message( AUTHOR_WARNING "+ Building for macOS." )
else()
	error( "Unknown operating system. Aborting." )
endif()

## ensure target folders exist
add_custom_target(SDCC_MAKE_BIN ALL COMMAND ${CMAKE_COMMAND} -E make_directory ${SDCC_PREFIX}/bin )
# add_custom_target(SDCC_MAKE_LIB ALL COMMAND ${CMAKE_COMMAND} -E make_directory ${SDCC_PREFIX}/lib )

ExternalProject_Add(
	sdcc
	SOURCE_DIR ${SDCC_SOURCE_DIR}
	CONFIGURE_COMMAND ${SDCC_SHELL} ${SDCC_BOOTSTRAP_SCRIPT}
	PREFIX ${SDCC_PREFIX}
	BUILD_COMMAND COMMAND cd ${SDCC_SOURCE_DIR} COMMAND make
	INSTALL_COMMAND COMMAND cd ${SDCC_SOURCE_DIR} COMMAND make install
	BUILD_IN_SOURCE 1
)

add_dependencies(sdcc SDCC_MAKE_BIN)

