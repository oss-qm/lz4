# ##########################################################################
# LZ4 programs - Makefile
# Copyright (C) Yann Collet 2011-2016
#
# GPL v2 License
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License along
# with this program; if not, write to the Free Software Foundation, Inc.,
# 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
#
# You can contact the author at :
#  - LZ4 homepage : http://www.lz4.org
#  - LZ4 source repository : https://github.com/lz4/lz4
# ##########################################################################
# fuzzer  : Test tool, to check lz4 integrity on target platform
# fuzzer32: Same as fuzzer, but forced to compile in 32-bits mode
# frametest  : Test tool, to check lz4frame integrity on target platform
# frametest32: Same as frametest, but forced to compile in 32-bits mode
# fullbench  : Precisely measure speed for each LZ4 function variant
# fullbench32: Same as fullbench, but forced to compile in 32-bits mode
# datagen : generates synthetic data samples for tests & benchmarks
# ##########################################################################

DESTDIR ?=
PREFIX  ?= /usr/local
VOID    := /dev/null
BINDIR  := $(PREFIX)/bin
MANDIR  := $(PREFIX)/share/man/man1
LZ4DIR  := ../lib
PRGDIR  := ../programs
TESTDIR := versionsTest
PYTHON  ?= python3

CFLAGS  ?= -O3   # can select custom flags. For example : CFLAGS="-O2 -g" make
CFLAGS  += -Wall -Wextra -Wundef -Wcast-qual -Wcast-align -Wshadow -Wswitch-enum \
           -Wdeclaration-after-statement -Wstrict-prototypes \
           -Wpointer-arith -Wstrict-aliasing=1
CFLAGS  += $(MOREFLAGS)
CPPFLAGS:= -I$(LZ4DIR) -DXXH_NAMESPACE=LZ4_
FLAGS   := $(CFLAGS) $(CPPFLAGS) $(LDFLAGS)


# Define *.exe as extension for Windows systems
ifneq (,$(filter Windows%,$(OS)))
EXT =.exe
else
EXT =
endif


# Default test parameters
TEST_FILES   := COPYING
FUZZER_TIME  := -T3mn
NB_LOOPS     ?= -i1


default: native

m32: fullbench32 fuzzer32 frametest32

native: fullbench fuzzer frametest datagen fasttest

all: native m32


lz4:
	$(MAKE) -C $(PRGDIR) $@

lz4c:
	$(MAKE) -C $(PRGDIR) $@

lz4c32:
	$(MAKE) -C $(PRGDIR) $@

fullbench  : $(LZ4DIR)/lz4.o $(LZ4DIR)/lz4hc.o $(LZ4DIR)/lz4frame.o $(LZ4DIR)/xxhash.o fullbench.c
	$(CC) $(FLAGS) $^ -o $@$(EXT)

fullbench32: $(LZ4DIR)/lz4.c $(LZ4DIR)/lz4hc.c $(LZ4DIR)/lz4frame.c $(LZ4DIR)/xxhash.c fullbench.c
	$(CC) -m32 $(FLAGS) $^ -o $@$(EXT)

fullbench-lib: fullbench.c $(LZ4DIR)/xxhash.c
	$(MAKE) -C $(LZ4DIR) liblz4.a
	$(CC) $(FLAGS) $^ -o $@$(EXT) $(LZ4DIR)/liblz4.a

fullbench-dll: fullbench.c $(LZ4DIR)/xxhash.c
	$(MAKE) -C $(LZ4DIR) liblz4
	$(CC) $(FLAGS) $^ -o $@$(EXT) -DLZ4_DLL_IMPORT=1 $(LZ4DIR)/liblz4.dll.a

fuzzer  : $(LZ4DIR)/lz4.o $(LZ4DIR)/lz4hc.o $(LZ4DIR)/xxhash.o fuzzer.c
	$(CC) $(FLAGS) $^ -o $@$(EXT)

fuzzer32: $(LZ4DIR)/lz4.c $(LZ4DIR)/lz4hc.c $(LZ4DIR)/xxhash.c fuzzer.c
	$(CC) -m32 $(FLAGS) $^ -o $@$(EXT)

frametest: $(LZ4DIR)/lz4frame.o $(LZ4DIR)/lz4.o $(LZ4DIR)/lz4hc.o $(LZ4DIR)/xxhash.o frametest.c
	$(CC) $(FLAGS) $^ -o $@$(EXT)

frametest32: $(LZ4DIR)/lz4frame.c $(LZ4DIR)/lz4.c $(LZ4DIR)/lz4hc.c $(LZ4DIR)/xxhash.c frametest.c
	$(CC) -m32 $(FLAGS) $^ -o $@$(EXT)

fasttest: $(LZ4DIR)/lz4.o fasttest.c
	$(CC) $(FLAGS) $^ -o $@$(EXT)

datagen : $(PRGDIR)/datagen.c datagencli.c
	$(CC) $(FLAGS) -I$(PRGDIR) $^ -o $@$(EXT)

clean:
	@$(MAKE) -C $(LZ4DIR) $@ > $(VOID)
	@$(RM) core *.o *.test tmp* \
        fullbench-dll$(EXT) fullbench-lib$(EXT) \
        fullbench$(EXT) fullbench32$(EXT) \
        fuzzer$(EXT) fuzzer32$(EXT) \
        frametest$(EXT) frametest32$(EXT) \
        fasttest$(EXT) datagen$(EXT)
	@rm -fR $(TESTDIR)
	@echo Cleaning completed

versionsTest:
	$(PYTHON) test-lz4-versions.py


#------------------------------------------------------------------------
#make install is validated only for Linux, OSX, kFreeBSD, Hurd and
#FreeBSD targets
ifneq (,$(filter $(shell uname),Linux Darwin GNU/kFreeBSD GNU FreeBSD))

test: test-lz4 test-lz4c test-fasttest test-frametest test-fullbench test-fuzzer

test32: test-lz4c32 test-frametest32 test-fullbench32 test-fuzzer32

test-all: test test32

test-lz4-sparse: lz4 datagen
	@echo "\n ---- test sparse file support ----"
	./datagen -g5M  -P100 > tmpSrc
	$(PRGDIR)/lz4 -B4D tmpSrc | $(PRGDIR)/lz4 -dv --sparse > tmpB4
	diff -s tmpSrc tmpB4
	$(PRGDIR)/lz4 -B5D tmpSrc | $(PRGDIR)/lz4 -dv --sparse > tmpB5
	diff -s tmpSrc tmpB5
	$(PRGDIR)/lz4 -B6D tmpSrc | $(PRGDIR)/lz4 -dv --sparse > tmpB6
	diff -s tmpSrc tmpB6
	$(PRGDIR)/lz4 -B7D tmpSrc | $(PRGDIR)/lz4 -dv --sparse > tmpB7
	diff -s tmpSrc tmpB7
	$(PRGDIR)/lz4 tmpSrc | $(PRGDIR)/lz4 -dv --no-sparse > tmpNoSparse
	diff -s tmpSrc tmpNoSparse
	ls -ls tmp*
	./datagen -s1 -g1200007 -P100 | $(PRGDIR)/lz4 | $(PRGDIR)/lz4 -dv --sparse > tmpOdd   # Odd size file (to generate non-full last block)
	./datagen -s1 -g1200007 -P100 | diff -s - tmpOdd
	ls -ls tmpOdd
	@$(RM) tmp*
	@echo "\n Compatibility with Console :"
	echo "Hello World 1 !" | $(PRGDIR)/lz4 | $(PRGDIR)/lz4 -d -c
	echo "Hello World 2 !" | $(PRGDIR)/lz4 | $(PRGDIR)/lz4 -d | cat
	echo "Hello World 3 !" | $(PRGDIR)/lz4 --no-frame-crc | $(PRGDIR)/lz4 -d -c
	@echo "\n Compatibility with Append :"
	./datagen -P100 -g1M > tmp1M
	cat tmp1M tmp1M > tmp2M
	$(PRGDIR)/lz4 -B5 -v tmp1M tmpC
	$(PRGDIR)/lz4 -d -v tmpC tmpR
	$(PRGDIR)/lz4 -d -v tmpC >> tmpR
	ls -ls tmp*
	diff tmp2M tmpR
	@$(RM) tmp*

test-lz4-contentSize: lz4 datagen
	@echo "\n ---- test original size support ----"
	./datagen -g15M > tmp
	$(PRGDIR)/lz4 -v tmp | $(PRGDIR)/lz4 -t
	$(PRGDIR)/lz4 -v --content-size tmp | $(PRGDIR)/lz4 -d > tmp2
	diff -s tmp tmp2
	# test large size [2-4] GB
	@./datagen -g3G -P100 | $(PRGDIR)/lz4 | $(PRGDIR)/lz4 --decompress --force --sparse - tmp
	@ls -ls tmp
	$(PRGDIR)/lz4 --quiet --content-size tmp | $(PRGDIR)/lz4 --verbose --decompress --force --sparse - tmp2
	@ls -ls tmp2
	@$(RM) tmp*

test-lz4-frame-concatenation: lz4 datagen
	@echo "\n ---- test frame concatenation ----"
	@echo -n > empty.test
	@echo hi > nonempty.test
	cat nonempty.test empty.test nonempty.test > orig.test
	@$(PRGDIR)/lz4 -zq empty.test > empty.lz4.test
	@$(PRGDIR)/lz4 -zq nonempty.test > nonempty.lz4.test
	cat nonempty.lz4.test empty.lz4.test nonempty.lz4.test > concat.lz4.test
	$(PRGDIR)/lz4 -d concat.lz4.test > result.test
	sdiff orig.test result.test
	@$(RM) *.test
	@echo frame concatenation test completed

test-lz4-multiple: lz4 datagen
	@echo "\n ---- test multiple files ----"
	@./datagen -s1        > tmp1 2> $(VOID)
	@./datagen -s2 -g100K > tmp2 2> $(VOID)
	@./datagen -s3 -g1M   > tmp3 2> $(VOID)
	$(PRGDIR)/lz4 -f -m tmp*
	ls -ls tmp*
	@$(RM) tmp1 tmp2 tmp3
	$(PRGDIR)/lz4 -df -m *.lz4
	ls -ls tmp*
	$(PRGDIR)/lz4 -f -m tmp1 notHere tmp2; echo $$?
	@$(RM) tmp*

test-lz4-basic: lz4 datagen
	@echo "\n ---- test lz4 basic compression/decompression ----"
	./datagen -g0     | $(PRGDIR)/lz4 -v     | $(PRGDIR)/lz4 -t
	./datagen -g16KB  | $(PRGDIR)/lz4 -9     | $(PRGDIR)/lz4 -t
	./datagen -g20KB > tmpSrc
	$(PRGDIR)/lz4 < tmpSrc | $(PRGDIR)/lz4 -d > tmpRes
	diff -q tmpSrc tmpRes
	$(PRGDIR)/lz4 --no-frame-crc < tmpSrc | $(PRGDIR)/lz4 -d > tmpRes
	diff -q tmpSrc tmpRes
	./datagen         | $(PRGDIR)/lz4        | $(PRGDIR)/lz4 -t
	./datagen -g6M -P99 | $(PRGDIR)/lz4 -9BD | $(PRGDIR)/lz4 -t
	./datagen -g17M   | $(PRGDIR)/lz4 -9v    | $(PRGDIR)/lz4 -qt
	./datagen -g33M   | $(PRGDIR)/lz4 --no-frame-crc | $(PRGDIR)/lz4 -t
	./datagen -g256MB | $(PRGDIR)/lz4 -vqB4D | $(PRGDIR)/lz4 -t
	@$(RM) tmp*

test-lz4-hugefile: lz4 datagen
	@echo "\n ---- test huge files compression/decompression ----"
	./datagen -g6GB   | $(PRGDIR)/lz4 -vB5D  | $(PRGDIR)/lz4 -qt
	./datagen -g6GB   | $(PRGDIR)/lz4 -v5BD  | $(PRGDIR)/lz4 -qt
	@$(RM) tmp*

test-lz4-testmode: lz4 datagen
	@echo "\n ---- bench mode ----"
	$(PRGDIR)/lz4 -bi1
	@echo "\n ---- test mode ----"
	./datagen | $(PRGDIR)/lz4 -t             && false || true
	./datagen | $(PRGDIR)/lz4 -tf            && false || true
	@echo "\n ---- pass-through mode ----"
	./datagen | $(PRGDIR)/lz4 -d  > $(VOID)  && false || true
	./datagen | $(PRGDIR)/lz4 -df > $(VOID)
	@echo "Hello World !" > tmp1
	$(PRGDIR)/lz4 -dcf tmp1
	@echo "from underground..." > tmp2
	$(PRGDIR)/lz4 -dcfm tmp1 tmp2
	@echo "\n ---- test cli ----"
	$(PRGDIR)/lz4     file-does-not-exist    && false || true
	$(PRGDIR)/lz4 -f  file-does-not-exist    && false || true
	$(PRGDIR)/lz4 -fm file1-dne file2-dne    && false || true
	$(PRGDIR)/lz4 -fm file1-dne file2-dne    && false || true

test-lz4: lz4 datagen test-lz4-basic test-lz4-multiple test-lz4-sparse \
          test-lz4-frame-concatenation test-lz4-testmode test-lz4-contentSize \
          test-lz4-hugefile

test-lz4c: lz4c datagen
	@echo "\n ---- test lz4c version ----"
	./datagen -g256MB | $(PRGDIR)/lz4c -l -v    | $(PRGDIR)/lz4c   -t

test-interop-32-64: lz4 lz4c32 datagen
	@echo "\n ---- test interoperability 32-bits -vs- 64 bits ----"
	./datagen -g16KB  | $(PRGDIR)/lz4c32 -9     | $(PRGDIR)/lz4    -t
	./datagen -P10    | $(PRGDIR)/lz4    -9B4   | $(PRGDIR)/lz4c32 -t
	./datagen         | $(PRGDIR)/lz4c32        | $(PRGDIR)/lz4    -t
	./datagen -g1M    | $(PRGDIR)/lz4    -3B5   | $(PRGDIR)/lz4c32 -t
	./datagen -g256MB | $(PRGDIR)/lz4c32 -vqB4D | $(PRGDIR)/lz4    -qt
	./datagen -g1G -P90 | $(PRGDIR)/lz4         | $(PRGDIR)/lz4c32 -t
	./datagen -g6GB   | $(PRGDIR)/lz4c32 -vq9BD | $(PRGDIR)/lz4    -qt

test-lz4c32-basic: lz4c32 datagen
	@echo "\n ---- test lz4c32 32-bits version ----"
	./datagen -g16KB  | $(PRGDIR)/lz4c32 -9     | $(PRGDIR)/lz4c32 -t
	./datagen         | $(PRGDIR)/lz4c32        | $(PRGDIR)/lz4c32 -t
	./datagen -g256MB | $(PRGDIR)/lz4c32 -vqB4D | $(PRGDIR)/lz4c32 -qt
	./datagen -g6GB   | $(PRGDIR)/lz4c32 -vqB5D | $(PRGDIR)/lz4c32 -qt

test-platform:
	@echo "\n ---- test lz4 $(QEMU_SYS) platform ----"
	$(QEMU_SYS) ./datagen -g16KB  | $(QEMU_SYS) $(PRGDIR)/lz4 -9     | $(QEMU_SYS) $(PRGDIR)/lz4 -t
	$(QEMU_SYS) ./datagen         | $(QEMU_SYS) $(PRGDIR)/lz4        | $(QEMU_SYS) $(PRGDIR)/lz4 -t
	$(QEMU_SYS) ./datagen -g256MB | $(QEMU_SYS) $(PRGDIR)/lz4 -vqB4D | $(QEMU_SYS) $(PRGDIR)/lz4 -qt
ifneq ($(QEMU_SYS),qemu-arm-static)
	$(QEMU_SYS) ./datagen -g3GB   | $(QEMU_SYS) $(PRGDIR)/lz4 -vqB5D | $(QEMU_SYS) $(PRGDIR)/lz4 -qt
endif

test-lz4c32: test-lz4c32-basic test-interop-32-64

test-fullbench: fullbench
	./fullbench --no-prompt $(NB_LOOPS) $(TEST_FILES)

test-fullbench32: fullbench32
	./fullbench32 --no-prompt $(NB_LOOPS) $(TEST_FILES)

test-fuzzer: fuzzer
	./fuzzer $(FUZZER_TIME)

test-fuzzer32: fuzzer32
	./fuzzer32 $(FUZZER_TIME)

test-frametest: frametest
	./frametest $(FUZZER_TIME)

test-frametest32: frametest32
	./frametest32 $(FUZZER_TIME)

test-fasttest: fasttest
	./fasttest

test-mem: lz4 datagen fuzzer frametest fullbench
	@echo "\n ---- valgrind tests : memory analyzer ----"
	valgrind --leak-check=yes --error-exitcode=1 ./datagen -g50M > $(VOID)
	./datagen -g16KB > tmp
	valgrind --leak-check=yes --error-exitcode=1 $(PRGDIR)/lz4 -9 -BD -f tmp $(VOID)
	./datagen -g16KB -s2 > tmp2
	./datagen -g16KB -s3 > tmp3
	valgrind --leak-check=yes --error-exitcode=1 $(PRGDIR)/lz4 --force --multiple tmp tmp2 tmp3
	./datagen -g16MB > tmp
	valgrind --leak-check=yes --error-exitcode=1 $(PRGDIR)/lz4 -9 -B5D -f tmp tmp2
	valgrind --leak-check=yes --error-exitcode=1 $(PRGDIR)/lz4 -t tmp2
	valgrind --leak-check=yes --error-exitcode=1 $(PRGDIR)/lz4 -bi1 tmp
	valgrind --leak-check=yes --error-exitcode=1 ./fullbench -i1 tmp tmp2
	./datagen -g256MB > tmp
	valgrind --leak-check=yes --error-exitcode=1 $(PRGDIR)/lz4 -B4D -f -vq tmp $(VOID)
	$(RM) tmp*
	valgrind --leak-check=yes --error-exitcode=1 ./fuzzer -i64 -t1
	valgrind --leak-check=yes --error-exitcode=1 ./frametest -i256

test-mem32: lz4c32 datagen
# unfortunately, valgrind doesn't seem to work with non-native binary...

endif
