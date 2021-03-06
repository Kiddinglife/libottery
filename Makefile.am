#  Libottery by Nick Mathewson.
#
#  This software has been dedicated to the public domain under the CC0
#  public domain dedication.
#
#  To the extent possible under law, the person who associated CC0 with
#  libottery has waived all copyright and related or neighboring rights
#  to libottery.
#
#  You should have received a copy of the CC0 legalcode along with this
#  work in doc/cc0.txt.  If not, see
#  <http://creativecommons.org/publicdomain/zero/1.0/>.

#    foreign -- this project doesn't need to have a GNU-style file layout.
#    1.9 -- require automake 1.9 or later
#    subdir-objects -- store object files in subdirectories corresponding
#        to the source that produced them.
AUTOMAKE_OPTIONS = foreign 1.9 subdir-objects

# Find macros in m4/. This is *not* redundant with AC_CONFIG_MACRO_DIR
# over in configure.ac, unfortunately.
ACLOCAL_AMFLAGS = -I m4

# VERSION_INFO defines the ABI version (aka 'soname') of the library
# we build.  Do not confuse this with the version number set by AC_INIT!
#
# To increment VERSION_INFO (current:revision:age):
#    If the ABI didn't change:
#        Return (current:revision+1:age)
#    If the ABI changed, but it's backward-compatible:
#        Return (current+1:0:age+1)
#    If the ABI changed and it isn't backward-compatible:
#        Return (current+1:0:0)
#
# History:
#    0:0:0 -- Libottery 0.0.0 (before the first releaes)
#
VERSION_INFO = 0:0:0

# Compiler and linker options to apply to everything.
# TODO: Make sure these all work
AM_CFLAGS = -Wall -W $(PTHREAD_CFLAGS) -I $(top_srcdir)/src

#####
# LDFLAGS that we want to add to all LDFLAG
GENERIC_LDFLAGS = -version-info $(VERSION_INFO) -no-undefined

#
# The library itself.
#

lib_LTLIBRARIES = libottery.la
libottery_la_LDFLAGS = $(GENERIC_LDFLAGS)
libottery_la_LIBADD = $(PTHREAD_LIBS)

# This code is always included in the library, regardless of build options.
libottery_la_SOURCES =				\
	src/chacha_merged.c			\
	src/ottery.c				\
	src/ottery_cpuinfo.c			\
	src/ottery_global.c			\
	src/ottery_entropy.c

# chacha_krovetz.c has to be built using special command-line options,
# and therefore must be put in its own "convenience library."
noinst_LTLIBRARIES =
if SIMD_CHACHA_1
noinst_LTLIBRARIES  += libchacha-simd1.la
libottery_la_LIBADD += libchacha-simd1.la
libchacha_simd1_la_SOURCES = src/chacha_krovetz.c
libchacha_simd1_la_CFLAGS  = $(AM_CFLAGS) $(SIMD1_CFLAGS) -DOTTERY_BUILDING_SIMD1
endif

# We may want to compile it twice with two different sets of options.
if SIMD_CHACHA_2
noinst_LTLIBRARIES  += libchacha-simd2.la
libottery_la_LIBADD += libchacha-simd2.la
libchacha_simd2_la_SOURCES = src/chacha_krovetz.c
libchacha_simd2_la_CFLAGS  = $(AM_CFLAGS) $(SIMD2_CFLAGS) -DOTTERY_BUILDING_SIMD2
endif

#
# Installed headers and other data.
#

include_HEADERS	=				\
	src/ottery.h				\
	src/ottery_common.h			\
	src/ottery_nolock.h			\
	src/ottery_st.h				\
	src/ottery_version.h

pkgconfigdir=$(libdir)/pkgconfig
pkgconfig_DATA  = libottery.pc

#
# Testing
#

# Default set of actual test cases.
TESTS = test/test_memclear test/test_shallow test/test_deep \
	test/test_vector_cmp.sh

# Programs to compile before running the tests
check_PROGRAMS = test/test_vectors test/bench_rng test/dump_bytes	\
		 test/test_memclear test/test_shallow test/test_deep

if ! WINDOWS
check_PROGRAMS += test/test_egd test/fake_egd
endif

# Data generated by test/test_vectors and by test/make_test_vectors.py.
# Also needs to exist before running the tests.
check_DATA =					\
	test/test_vectors.expected		\
	test/test_vectors.actual		\
	test/test_vectors.actual-nosimd		\
	test/test_vectors.actual-midrange

# The python script that generates test/test_vectors.expected
check_SCRIPTS = test/make_test_vectors.py

test_test_vectors_SOURCES = test/test_vectors.c test/streams.c
test_test_vectors_LDADD = libottery.la $(PTHREAD_LIBS)

# TODO: skip this test if libcrypto is unavailable
test_bench_rng_SOURCES = test/bench_rng.c
test_bench_rng_LDADD = libottery.la -lcrypto $(PTHREAD_LIBS)

test_dump_bytes_SOURCES = test/dump_bytes.c
test_dump_bytes_LDADD = libottery.la $(PTHREAD_LIBS)

test_test_memclear_SOURCES = test/test_memclear.c
test_test_memclear_LDADD = libottery.la $(PTHREAD_LIBS)

test_test_shallow_SOURCES = test/test_shallow.c test/tinytest.c
test_test_shallow_LDADD = libottery.la $(PTHREAD_LIBS)

test_test_deep_SOURCES = test/test_deep.c test/tinytest.c
test_test_deep_LDADD = libottery.la $(PTHREAD_LIBS)

if ! WINDOWS
test_test_egd_SOURCES = test/test_egd.c
test_test_egd_LDADD = libottery.la $(PTHREAD_LIBS)

# How to build our EGD testing tool and our fake egd implementation.
test_fake_egd_SOURCES = test/fake_egd.c
endif

# TODO: Skip when Python is unavailable.
test/test_vectors.expected: $(top_srcdir)/test/make_test_vectors.py
	$(AM_V_GEN)$(PYTHON) $(top_srcdir)/test/make_test_vectors.py > test/test_vectors.expected

test/test_vectors.actual: test/test_vectors$(EXEEXT)
	$(AM_V_GEN)./test/test_vectors > test/test_vectors.actual

test/test_vectors.actual-midrange: test/test_vectors$(EXEEXT)
	$(AM_V_GEN)./test/test_vectors midrange > test/test_vectors.actual-midrange

test/test_vectors.actual-nosimd: test/test_vectors$(EXEEXT)
	$(AM_V_GEN)./test/test_vectors no-simd > test/test_vectors.actual-nosimd

#####
# If we have a haskell, we can run our "Spec" tests.
if USEGHC

# This C program behaves deterministically in a way that we reimplement
# in Haskell, to make sure that our RNG is following the spec.
check_PROGRAMS += test/test_spec
test_test_spec_SOURCES = test/test_spec.c
test_test_spec_LDADD = libottery.la $(PTHREAD_LIBS)

# More generated info that we'll need to make sure we generate to run
# our unit tests.
check_DATA += 				\
	test/hs/test_ottery.output 	\
	test/test_spec.output

# How to build test_ottery, the Haskell test implementation of the
# libottery spec
test/hs/test_ottery: test/hs/Ottery.hs test/hs/ChaCha.hs test/hs/test_ottery.hs
	$(AM_V_GEN)$(MKDIR_P) ./test/hs
	$(AM_V_AT)$(GHC) -o test/hs/test_ottery \
		-odir ./test/hs -hidir ./test/hs \
		$(top_srcdir)/test/hs/Ottery.hs \
		$(top_srcdir)/test/hs/ChaCha.hs \
		$(top_srcdir)/test/hs/test_ottery.hs

# Rule to generate the expected output from the haskell code
test/hs/test_ottery.output: test/hs/test_ottery test/test_spec
	$(AM_V_GEN)./test/hs/test_ottery `./test/test_spec --blocks-per-call` > test/hs/test_ottery.output

# Rule to generate the actual output which should match the haskell code.
test/test_spec.output: test/test_spec test/test_spec_seed
	$(AM_V_GEN)./test/test_spec $(top_srcdir)/test/test_spec_seed > test/test_spec.output

TESTS += test/test_spec_cmp.sh

endif

#####
# If we have a python, we can run the EGD tests.
if USEPYTHON
if ! WINDOWS
TESTS += test/test_egd.py
endif
endif

#
# Miscellaneous
#

# Internal-use headers
noinst_HEADERS = 			\
	src/chacha_merged_ecrypt.h 	\
	src/ottery-internal.h 		\
	src/ottery-threading.h 		\
	src/ottery_entropy_cryptgenrandom.c	\
	src/ottery_entropy_egd.c	\
	src/ottery_entropy_rdrand.c	\
	src/ottery_entropy_urandom.c	\
	test/st_wrappers.h 		\
	test/streams.h 			\
	test/tinytest.h 		\
	test/tinytest_macros.h

# Things we need to distribute, not listed elsewhere.
EXTRA_DIST =					\
	test/make_test_vectors.py		\
	test/test_egd.py			\
	test/test_vector_cmp.sh			\
	test/test_spec_cmp.sh			\
	test/test_spec_seed			\
	test/hs/Ottery.hs			\
	test/hs/ChaCha.hs			\
	test/hs/test_ottery.hs			\
	etc/doxygen.conf			\
	etc/uncrustify.cfg			\
	m4/ottery_local.m4			\
	COPYING					\
	README.md				\
	TODO

# Files to remove on 'make clean', not listed elsewhere.
CLEANFILES = $(noinst_DATA) test/hs/*.hi test/hs/*.o test/hs/test_ottery \
	test/test_vectors.expected \
	test/test_vectors.actual \
	test/test_vectors.actual-nosimd \
	test/test_vectors.actual-midrange \
	test/hs/test_ottery.output \
	test/test_spec.output \
	*.gcov src/*.gcov test/*.gcov \
	*.gcno src/*.gcno test/*.gcno \
	*.gcda src/*.gcda test/*.gcda

# Files to clean up with uncrustify.
UNCRUSTIFY_FILES =				\
		src/ottery.c			\
		src/ottery_cpuinfo.c		\
		src/ottery_global.c		\
		src/ottery_entropy.c		\
		$(include_HEADERS)		\
		src/chacha_merged_ecrypt.h 	\
		src/ottery-internal.h 		\
		test/st_wrappers.h 		\
		test/streams.h 			\
		test/dump_bytes.c		\
		test/streams.c			\
		test/test_deep.c		\
		test/test_memclear.c		\
		test/test_shallow.c		\
		test/test_spec.c		\
		test/test_vectors.c

# Ignoring these, because they're not ours:
#		src/chacha_merged.c
#		test/tinytest.h
#		test/tinytest_macros.h
# 		src/chacha_krovetz.c

uncrustify:
	uncrustify -c etc/uncrustify.cfg --replace -l C $(UNCRUSTIFY_FILES)

# Rebuild the doxygen documentation
doxygen:
	doxygen etc/doxygen.conf

# Generate test coverage for the entire library.  Assumes gcov+gcc.
# Probably doesn't work in a VPATH build.
COVERAGE_CFLAGS = -fprofile-arcs -ftest-coverage
coverage:
	$(MAKE) clean check CFLAGS="$(CFLAGS) $(COVERAGE_CFLAGS)"
	gcov -o src/.libs $(libottery_la_SOURCES)
if SIMD_CHACHA_1
	gcov -o src/.libs/libchacha_simd1_la-chacha_krovetz.o \
	    src/chacha_krovetz.c && \
	mv -f chacha_krovetz.c.gcov chacha_krovetz_simd1.c.gcov
endif
if SIMD_CHACHA_2
	gcov -o src/.libs/libchacha_simd2_la-chacha_krovetz.o \
	    src/chacha_krovetz.c && \
	mv -f chacha_krovetz.c.gcov chacha_krovetz_simd2.c.gcov
endif

