---
title: Platform Benchmark
date: 2022-09-07 22:14:02
categories:
- benchmark
tags:
- benchmark
---
## LMbench

Source Code : http://www.bitmover.com/lmbench/lmbench.html

Compile Command ： 

```shell
chmod 777 -R scripts/
make CC=arm-linux-gnueabihf-gcc OS=arm-linux
```

Executable File Path ： 

```shell
#cd bin/$(OS)/
cd bin/arm-linux
```

Execute Command ： 

```shell
./lat_mem_rd -P 1 8 32
```

## CoreMark

Source Code : https://www.eembc.org/coremark/index.php

Porting Step :

* Make a new directory  for new platform.

* Copy "linux/" to new platform folder.
* Modify the "CC" and "PORT_CFLAGS" in  "core_portme.mak".
* Modify "MULTITHREAD" and "USE_FORK" Maco in "core_portme.h" if you want to use multiple thread.

For Example Porting to CA53 for smp : 

```shell
mkdir port
mkdir port/linux_arm_ca53_mt2
vi port/linux_arm_ca53_mt2/core_port.mak
#Modify CC=arm-linux-gnueabihf-gcc
#Modify PORT_CFLAGS = -mcpu=cortex-a53 -mfpu=neon -static -O3
vi port/linux_arm_ca53_mt2/core_port.h
#Modify #define MULTITHREAD 1 => #define MULTITHREAD 2
#Modify #define USE_FORK 0 => #define USE_FORK 1
```

Compile Command : 

```shell
make PORT_DIR=port/linux_arm_ca53_mt2/
```

Executable File Path :

```shell
./coremark.exe
```

Execute Command :

```shell
./coremark.exe
```

## Stream

Source Code : 

```shell
wget -r -np -nH --cut-dirs=3 -R index.html http://www.cs.virginia.edu/stream/FTP/Code/
```

Porting Step :

* Modify Makefile to support arm gnu.
* Add compile attribute for arm gnu.

For Example :

```shell
# Makefile
CROSS_COMPILE = arm-linux-gnueabihf-
CC = $(CROSS_COMPILE)gcc
CFLAGS = -O -static -DSTREAM_ARRAY_SIZE=0x100000 -DNTIMES=20

ifeq ($(PLATFORM),ca7)
CFLAGS += -mcpu=cortex-a7 -mfpu=neon
else ifeq ($(PLATFORM),ca53)
CFLAGS += -mcpu=cortex-a53 -mfpu=neon
else ifeq ($(PLATFORM),rv32)
CROSS_COMPILE = riscv32-linux-
FLAGS += -march=rv32im -mabi=ilp32
else
CFLAGS += -mcpu=cortex-a9 -mfpu=neon
endif

#FF = $(CROSS_COMPILE)g77
FF = $(CROSS_COMPILE)gfortran

FFLAGS = -O2 -static

all: stream_$(PLATFORM) stream_omp_$(PLATFORM)
    
stream_fortran_$(PLATFORM): stream.f mysecond.o
	$(CC) $(CFLAGS) -c mysecond.c
	$(FF) $(FFLAGS) -c stream.f
	$(FF) $(FFLAGS) stream.o mysecond.o -o $@

stream_$(PLATFORM): stream.c
	$(CC) $(CFLAGS) stream.c -o $@ 
	@$(CROSS_COMPILE)strip --strip-unneeded $@

stream_omp_$(PLATFORM): stream.c
	$(CC) $(CFLAGS) -fopenmp stream.c -o $@
	@$(CROSS_COMPILE)strip --strip-unneeded $@

clean:
	rm -f stream_fortran_$(PLATFORM) stream_$(PLATFORM) stream_omp_$(PLATFORM) *.o

# an example of a more complex build line for the Intel icc compiler
stream.icc: stream.c
	icc -O3 -xCORE-AVX2 -ffreestanding -qopenmp -DSTREAM_ARRAY_SIZE=80000000 -DNTIMES=20 stream.c -o stream.omp.AVX2.80M.20x.icc

#code
#before:
static STREAM_TYPE 	c[STREAM_ARRAY_SIZE+OFFSET];
static STREAM_TYPE 	b[STREAM_ARRAY_SIZE+OFFSET];
static STREAM_TYPE 	a[STREAM_ARRAY_SIZE+OFFSET];
#after:
static STREAM_TYPE 	__attribute__ ((aligned (4096))) c[STREAM_ARRAY_SIZE+OFFSET];
static STREAM_TYPE 	__attribute__ ((aligned (4096))) b[STREAM_ARRAY_SIZE+OFFSET];
static STREAM_TYPE 	__attribute__ ((aligned (4096))) a[STREAM_ARRAY_SIZE+OFFSET];
```

Compile Command : 

```shell
make PLATFORM=ca53
```

Executable File Path ：

```shell
./stream_ca53
./stream_omp_ca53
```

Execute Command ：

```shell
#multiple processor
./stream_omp_ca53
#single processor
./stream_ca53
```

## Whetstone

Source Code : http://www.netlib.org/benchmark/whetstone.c

Porting step :

* Create Makefile

```makefile
CC=$(CROSS_COMPILE)gcc

OPTIMIZE = -O0    # Optimization Level

CFLAGS = $(OPTIMIZE) -static
LD = -lm

OUTPUT = whetstone

SRC = whetstone.c

all:$(SRC)
	$(CC) $(CFLAGS) $(SRC) $(LFLAGAS) -o $(OUTPUT) $(LD)

clean:
	-rm -f *.o $(OUTPUT)
```

Compile Command : 

```shell
make
```

Executable File Path ：

```shell
./whetstone
```

Execute Command ：

```shell
./whetstone 10000
```



