cmake_minimum_required(VERSION 2.8)
project (libpam)

if (DEFINED CMAKE_TOOLCHAIN_FILE)
	message("Include Cross Platform cmake: " ${CROSS_PLATFORM_CMAKE})
	get_filename_component(COMPILER_NAME ${CMAKE_C_COMPILER} NAME)
	message( "CMAKE_C_COMPILER: ${COMPILER_NAME}")
	string(REGEX REPLACE "(.*)-gcc" "\\1" target_platform "${COMPILER_NAME}")
	message( "target_platform: ${target_platform}")
	set(target_platform_preconfigure_options "CC=${target_platform}-gcc;AR=${target_platform}-ar;LD=${target_platform}-ld;RANLIB=${target_platform}-ranlib;NM=${target_platform}-nm;AS=${target_platform}-as;OBJDUMP=${target_platform}-objdump;OBJCOPY=${target_platform}-objcopy;STRIP=${target_platform}-strip;STRINGS=${target_platform}-strings;SIZE=${target_platform}-size")
endif(DEFINED CMAKE_TOOLCHAIN_FILE)

set(SELF_CPPFLAGS "")
set(SELF_CFLAGS "")
set(SELF_LDFLAGS "")
set(SELF_DEPENDS "")

if(NOT TARGET build_libfl)
	add_subdirectory(libfl-cmake)
endif(NOT TARGET build_libfl)
set(SELF_DEPENDS "build_libfl")
set(SELF_CFLAGS "-I${CMAKE_BINARY_DIR}/include")
set(SELF_CPPFLAGS "-I${CMAKE_BINARY_DIR}/include")
set(SELF_LDFLAGS "-L${CMAKE_BINARY_DIR}/lib")

set(TOOL_LIBPAM_SOURCE Linux-PAM-1.1.6.tar.bz2) 
set(TOOL_LIBPAM_DIR Linux-PAM-1.1.6)
set(CONFIG_ADD_OPTIONS "--includedir=${CMAKE_BINARY_DIR}/include/security")
	if(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_LIBPAM
			# get source
			#COMMAND wget -qO- https://fedorahosted.org/releases/l/i/linux-pam/${TOOL_LIBPAM_SOURCE} | tar -xjvf -
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBPAM_SOURCE} || wget https://fedorahosted.org/releases/l/i/linux-pam/${TOOL_LIBPAM_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBPAM_SOURCE}
			COMMAND tar -xjvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBPAM_SOURCE}
			# configure: not support outside config
			COMMAND cd ${TOOL_LIBPAM_DIR} && CFLAGS="${SELF_CFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" LDFLAGS="${SELF_LDFLAGS}" ./configure --host="${target_platform}" --prefix=${CMAKE_BINARY_DIR} ${CONFIG_ADD_OPTIONS} 
			# build
			COMMAND cd ${TOOL_LIBPAM_DIR} && make -j4 && make install
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT "Build libpam"
		)
	else(DEFINED CMAKE_TOOLCHAIN_FILE)
		add_custom_command(
			OUTPUT BUILD_LIBPAM
			# get source
			#COMMAND wget -qO- https://fedorahosted.org/releases/l/i/linux-pam/${TOOL_LIBPAM_SOURCE} | tar -xjvf - 
			COMMAND test -e ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBPAM_SOURCE} || wget https://fedorahosted.org/releases/l/i/linux-pam/${TOOL_LIBPAM_SOURCE} -O ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBPAM_SOURCE}
			COMMAND tar -xjvf ${CMAKE_CURRENT_SOURCE_DIR}/${TOOL_LIBPAM_SOURCE}
			# configure
			COMMAND cd ${TOOL_LIBPAM_DIR} && ./configure --prefix=${CMAKE_BINARY_DIR} LDFLAGS="${SELF_LDFLAGS}" CPPFLAGS="${SELF_CPPFLAGS}" ${CONFIG_ADD_OPTIONS}
			# build
			COMMAND cd ${TOOL_LIBPAM_DIR} && make -j4 && make install 
			WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
			COMMENT "Build libpam" 
		)
	endif(DEFINED CMAKE_TOOLCHAIN_FILE)
	add_custom_target(build_libpam ALL DEPENDS ${SELF_DEPENDS} BUILD_LIBPAM )
