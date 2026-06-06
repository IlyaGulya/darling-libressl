#
# Copyright (c) 2014 Brent Cook
#
# Permission to use, copy, modify, and distribute this software for any
# purpose with or without fee is hereby granted, provided that the above
# copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
# WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
# ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
# WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
# ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
# OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

set(
	SSL_SRC
	bio_ssl.c
	d1_both.c
	d1_lib.c
	d1_pkt.c
	d1_srtp.c
	pqueue.c
	s3_cbc.c
	s3_lib.c
	ssl_asn1.c
	ssl_both.c
	ssl_cert.c
	ssl_ciph.c
	ssl_ciphers.c
	ssl_clnt.c
	ssl_err.c
	ssl_init.c
	ssl_kex.c
	ssl_lib.c
	ssl_methods.c
	ssl_pkt.c
	ssl_rsa.c
	ssl_seclevel.c
	ssl_sess.c
	ssl_sigalgs.c
	ssl_srvr.c
	ssl_stat.c
	ssl_tlsext.c
	ssl_transcript.c
	ssl_txt.c
	ssl_versions.c
	t1_enc.c
	t1_lib.c
	tls_buffer.c
	tls_content.c
	tls_key_share.c
	tls_lib.c
	tls12_key_schedule.c
	tls12_lib.c
	tls12_record_layer.c
	tls13_client.c
	tls13_error.c
	tls13_handshake.c
	tls13_handshake_msg.c
	tls13_key_schedule.c
	tls13_legacy.c
	tls13_lib.c
	tls13_quic.c
	tls13_record.c
	tls13_record_layer.c
	tls13_server.c
)

set(
	BS_SRC
	bs_ber.c
	bs_cbb.c
	bs_cbs.c
)

add_library(ssl_obj OBJECT ${SSL_SRC})
make_fat(ssl_obj)
target_include_directories(ssl_obj BEFORE
	PRIVATE
		hidden
		.
		../crypto
		../crypto/bio
		../include/compat
		../include
		${CMAKE_BINARY_DIR}/include)

if(MSVC)
	# "C4702" - unreachable code
	set_source_files_properties(d1_pkt.c s3_cbc.c PROPERTIES
		COMPILE_OPTIONS /wd4702
	)
endif()

add_library(bs_obj OBJECT ${BS_SRC})
make_fat(bs_obj)
target_include_directories(bs_obj BEFORE
	PRIVATE
		.
		../include/compat)

if(BUILD_SHARED_LIBS)
	set(DYLIB_INSTALL_NAME "/usr/lib/libssl.46.dylib")
	set(DYLIB_CURRENT_VERSION "47.0.0")
	set(DYLIB_COMPAT_VERSION "47.0.0")
	if(DARLING)
		add_darling_library(ssl ${SSL_SRC} ${BS_SRC} ${LIBRESSL_COMPAT_SRC})
	else()
		add_darling_library(ssl $<TARGET_OBJECTS:ssl_obj> $<TARGET_OBJECTS:compat_obj> $<TARGET_OBJECTS:bs_obj>)
	endif()
else()
	set(DYLIB_INSTALL_NAME "/usr/lib/libssl.46.dylib")
	set(DYLIB_CURRENT_VERSION "47.0.0")
	set(DYLIB_COMPAT_VERSION "47.0.0")
	if(DARLING)
		add_darling_library(ssl ${SSL_SRC} ${BS_SRC} ${LIBRESSL_COMPAT_SRC} empty.c)
	else()
		add_darling_library(ssl $<TARGET_OBJECTS:ssl_obj> empty.c)
	endif()
endif()
target_include_directories(ssl BEFORE
	PRIVATE
		hidden
		.
		../crypto
		../crypto/bio
		../include/compat
		../include
		${CMAKE_BINARY_DIR}/include)

export_symbol(ssl ${CMAKE_CURRENT_SOURCE_DIR}/ssl.sym)
target_link_libraries(ssl PRIVATE crypto system ${PLATFORM_LIBS})
set_target_properties(ssl PROPERTIES
	OUTPUT_NAME ssl.46
	ARCHIVE_OUTPUT_NAME ssl.46
	EXPORT_NAME SSL
	VERSION ${SSL_VERSION}
	SOVERSION ${SSL_MAJOR_VERSION}
)
if(NOT CMAKE_VERSION VERSION_LESS 3.27.0)
	set_target_properties(ssl PROPERTIES DLL_NAME_WITH_SOVERSION TRUE)
endif()

target_include_directories(
	ssl
	PUBLIC
	$<BUILD_INTERFACE:${CMAKE_BINARY_DIR}/include>
	$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(HOST_AARCH64)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/aarch64/)
	target_include_directories(ssl PRIVATE ../crypto/arch/aarch64/)
elseif(HOST_ARM)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/arm/)
	target_include_directories(ssl PRIVATE ../crypto/arch/arm/)
elseif(HOST_I386)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/i386/)
	target_include_directories(ssl PRIVATE ../crypto/arch/i386/)
elseif(HOST_LOONGARCH64)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/loongarch64)
	target_include_directories(ssl PRIVATE ../crypto/arch/loongarch64)
elseif(HOST_MIPS64)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/mips64)
	target_include_directories(ssl PRIVATE ../crypto/arch/mips64)
elseif(HOST_MIPS)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/mips)
	target_include_directories(ssl PRIVATE ../crypto/arch/mips)
elseif(HOST_POWERPC)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/powerpc)
	target_include_directories(ssl PRIVATE ../crypto/arch/powerpc)
elseif(HOST_POWERPC64)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/powerpc64)
	target_include_directories(ssl PRIVATE ../crypto/arch/powerpc64)
elseif(HOST_RISCV64)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/riscv64)
	target_include_directories(ssl PRIVATE ../crypto/arch/riscv64)
elseif(HOST_SPARC64)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/sparc64)
	target_include_directories(ssl PRIVATE ../crypto/arch/sparc64)
elseif(HOST_X86_64)
	target_include_directories(ssl_obj PRIVATE ../crypto/arch/amd64)
	target_include_directories(ssl PRIVATE ../crypto/arch/amd64)
endif()

if(ENABLE_LIBRESSL_INSTALL)
	install(
		TARGETS ssl
		EXPORT SSL-target
	)
	export(
		EXPORT SSL-target
		FILE "${LibreSSL_BINARY_DIR}/LibreSSL-SSL.cmake"
		NAMESPACE LibreSSL::
	)
	install(
		TARGETS ssl
		ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
		LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
		RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
	)
	install(
		EXPORT SSL-target
		FILE "LibreSSL-SSL.cmake"
		NAMESPACE LibreSSL::
		DESTINATION "${LIBRESSL_INSTALL_CMAKEDIR}"
	)
endif(ENABLE_LIBRESSL_INSTALL)

# build static library for regression test
if(BUILD_SHARED_LIBS)
	add_library(ssl-static STATIC $<TARGET_OBJECTS:ssl_obj>)
	target_link_libraries(ssl-static crypto-static ${PLATFORM_LIBS})
endif()
