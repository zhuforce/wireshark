# CMakeLists.txt
#
# Wireshark - Network traffic analyzer
# By Gerald Combs <gerald@wireshark.org>
# Copyright 1998 Gerald Combs
#
# SPDX-License-Identifier: GPL-2.0-or-later
#

include(UseABICheck)

set(WIRETAP_PUBLIC_HEADERS
	file_wrappers.h
	merge.h
	pcap-encap.h
	pcapng_module.h
	wtap.h
	wtap_opttypes.h
)

set(WIRETAP_NONGENERATED_FILES
	5views.c
	aethra.c
	ascendtext.c
	atm.c
	ber.c
	btsnoop.c
	camins.c
	capsa.c
	catapult_dct2000.c
	commview.c
	cosine.c
	csids.c
	daintree-sna.c
	dbs-etherwatch.c
	dpa400.c
	dct3trace.c
	erf.c
	eyesdn.c
	file_access.c
	file_wrappers.c
	hcidump.c
	i4btrace.c
	ipfix.c
	iptrace.c
	iseries.c
	json.c
	k12.c
	lanalyzer.c
	libpcap.c
	logcat.c
	logcat_text.c
	merge.c
	mpeg.c
	mplog.c
	mime_file.c
	mp2t.c
	netmon.c
	netscaler.c
	netscreen.c
	nettl.c
	nettrace_3gpp_32_423.c
	network_instruments.c
	netxray.c
	ngsniffer.c
	packetlogger.c
	pcap-common.c
	pcapng.c
	peekclassic.c
	peektagged.c
	pem.c
	pppdump.c
	radcom.c
	ruby_marshal.c
	snoop.c
	stanag4607.c
	tnef.c
	toshiba.c
	visual.c
	vms.c
	vwr.c
	wtap.c
	wtap_opttypes.c
	${CMAKE_SOURCE_DIR}/version_info.c
)

set(WIRETAP_FILES ${WIRETAP_NONGENERATED_FILES})

add_lex_files(LEX_FILES WIRETAP_FILES
	ascend_scanner.l
	k12text.l
)

add_yacc_files(YACC_FILES WIRETAP_FILES
	ascend.y
)

#
# All files are warning-clean.  (Let's keep it that way.)
#
set_source_files_properties(
	${WIRETAP_FILES}
	PROPERTIES
	COMPILE_FLAGS "${WERROR_COMMON_FLAGS}"
)

set(wiretap_LIBS
	${GLIB2_LIBRARIES}
	${GMODULE2_LIBRARIES}
	${ZLIB_LIBRARIES}
	wsutil
)

add_library(wiretap
	${WIRETAP_FILES}
	${CMAKE_BINARY_DIR}/image/wiretap.rc
)

add_dependencies(wiretap version)

set_target_properties(wiretap PROPERTIES
	PREFIX "lib"
	COMPILE_DEFINITIONS "WS_BUILD_DLL"
	LINK_FLAGS "${WS_LINK_FLAGS}"
	VERSION "0.0.0" SOVERSION 0
	FOLDER "DLLs"
)

if(ENABLE_APPLICATION_BUNDLE)
	if(NOT CMAKE_CFG_INTDIR STREQUAL ".")
		# Xcode
		set_target_properties(wiretap PROPERTIES
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/run/$<CONFIG>/Wireshark.app/Contents/Frameworks
		)
	else()
		set_target_properties(wiretap PROPERTIES
			LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/run/Wireshark.app/Contents/Frameworks
		)
	endif()
endif()

ABICHECK(libwiretap)

add_custom_command(OUTPUT libwiretap.abi.tar.gz
	COMMAND ${CMAKE_COMMAND} -E remove_directory ${ABICHECK_TMPDIR}
	COMMAND ${CMAKE_COMMAND} -E make_directory ${ABICHECK_TMPDIR}
	COMMAND ${ABI_COPY_COMMAND} ../config.h ${ABICHECK_TMPDIR}/ ${ABI_COPY_FLAGS}
	COMMAND ${ABI_COPY_COMMAND} ${ABICHECK_HEADERS} ${ABICHECK_TMPDIR}/ ${ABI_COPY_FLAGS}
	COMMAND ${ABICHECK_COMMAND}
	COMMAND cp ${CMAKE_CURRENT_BINARY_DIR}/abi_dumps/libwiretap/libwiretap_* ${CMAKE_CURRENT_BINARY_DIR}/libwiretap.abi.tar.gz
	COMMAND rm -rf ${ABICHECK_TMPDIR} ${CMAKE_CURRENT_BINARY_DIR}/abi_dumps
	DEPENDS ${HEADERS} wiretap)

target_link_libraries(wiretap ${wiretap_LIBS})

install(TARGETS wiretap
	LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	RUNTIME DESTINATION ${CMAKE_INSTALL_LIBDIR}
	ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

if(NOT WIN32)
	install(FILES ${WIRETAP_PUBLIC_HEADERS}
		DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/${CPACK_PACKAGE_NAME}/wiretap"
	)
endif()

CHECKAPI(
	NAME
	  wiretap
	SWITCHES
## 'abort' checking disabled for now pending resolution of existing use of g_assert & g_error
##	  -g abort -g termoutput
	  -g termoutput -build
	SOURCES
	  ${WIRETAP_NONGENERATED_FILES}
# LEX files commented out due to use of malloc, free etc.
#	  ${LEX_FILES}
	  ${YACC_FILES}
)

#
# Editor modelines  -  http://www.wireshark.org/tools/modelines.html
#
# Local variables:
# c-basic-offset: 8
# tab-width: 8
# indent-tabs-mode: t
# End:
#
# vi: set shiftwidth=8 tabstop=8 noexpandtab:
# :indentSize=8:tabSize=8:noTabs=false:
#
