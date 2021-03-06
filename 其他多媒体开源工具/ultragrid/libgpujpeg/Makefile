#
# Copyright (c) 2011, CESNET z.s.p.o
# Copyright (c) 2011, Silicon Genome, LLC.
#
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#     * Redistributions of source code must retain the above copyright
#       notice, this list of conditions and the following disclaimer.
#
#     * Redistributions in binary form must reproduce the above copyright
#       notice, this list of conditions and the following disclaimer in the
#       documentation and/or other materials provided with the distribution.
# 
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
#

# Shared/Static library?
SHARED_LIBRARY ?= 0
# Use constant memory for huffman coder tables?
HUFFMAN_CODER_TABLES_IN_CONSTANT ?= 1
# Use DCT from NPP library (is faster but causes color change)
# If set to 0, own implementation is used, it doesn't cause color change
# but for non-fermi GPUs it is slower
DCT_FROM_NPP ?= 1
# Flag if use OpenGL
USE_OPENGL ?= 0
# Debug
DEBUG ?= 0

# Target executable
TARGET := libgpujpeg.a
ifeq ($(SHARED_LIBRARY),1)
    TARGET := libgpujpeg.so
endif

# C files
CFILES := \
    gpujpeg_common.c \
    gpujpeg_encoder.c \
    gpujpeg_decoder.c \
    gpujpeg_table.c \
    gpujpeg_dct_cpu.c \
    gpujpeg_huffman_cpu_encoder.c \
    gpujpeg_huffman_cpu_decoder.c \
    gpujpeg_writer.c \
    gpujpeg_reader.c
# CUDA files
CUFILES := \
    gpujpeg_dct_gpu.cu \
    gpujpeg_preprocessor.cu \
    gpujpeg_huffman_gpu_encoder.cu \
    gpujpeg_huffman_gpu_decoder.cu

# Common flags
COMMONFLAGS += -I. -O2 -g

# Compilers
CC := gcc -fPIC
LINK := g++ -fPIC
ifdef CUDA_INSTALL_PATH
	NVCC := $(CUDA_INSTALL_PATH)/bin/nvcc -Xcompiler -fPIC
	COMMONFLAGS += -I$(CUDA_INSTALL_PATH)/include
else
	NVCC := nvcc
endif


# Ultragrid changes
ARCH := $(shell uname -s)
ifeq ($(ARCH),Darwin)
    COMMONFLAGS += -m32 -DHAVE_MACOSX
endif

# C flags
CFLAGS += $(COMMONFLAGS) -std=c99
# CUDA flags
NVCCFLAGS += $(COMMONFLAGS) \
	-gencode arch=compute_20,code=sm_20 \
	-gencode arch=compute_13,code=sm_13 \
	-gencode arch=compute_11,code=sm_11 \
	-gencode arch=compute_10,code=sm_10 \

NVCC_VERSION_MAJOR := $(shell $(NVCC) --version |grep release|sed  's/^.*release \([0-9][0-9]*\).*$$/\1/')
NVCC_VERSION_MINOR := $(shell $(NVCC) --version |grep release|sed  's/^.*release [0-9][0-9]*\.\([0-9][0-9]*\).*$$/\1/')
NVCC_VERSION_GE_4_2 := $(shell test $(NVCC_VERSION_MAJOR) -ge 4 -a $(NVCC_VERSION_MINOR) -ge 2 && echo yes)

ifeq ($(NVCC_VERSION_GE_4_2),yes)
NVCCFLAGS += \
	-gencode arch=compute_30,code=sm_30 
endif


# Linker flags
LDFLAGS += -shared

# Debug
ifeq ($(DEBUG),1)
    COMMONFLAGS += -g -D_DEBUG
endif    

# Other Flags
ifeq ($(HUFFMAN_CODER_TABLES_IN_CONSTANT),1)
    COMMONFLAGS += -DGPUJPEG_HUFFMAN_CODER_TABLES_IN_CONSTANT
endif
ifeq ($(DCT_FROM_NPP),1)
    COMMONFLAGS += -DGPUJPEG_DCT_FROM_NPP
endif
ifeq ($(USE_OPENGL),1)
    LDFLAGS += -lGLEW
    COMMONFLAGS += -DGPUJPEG_USE_OPENGL
endif 

# Do 32bit vs. 64bit setup
LBITS := $(shell getconf LONG_BIT)
ifeq ($(LBITS),64)
    # 64bit
    CUDA_LIB = -L$(CUDA_INSTALL_PATH)/lib64
else
    # 32bit
    CUDA_LIB = -L$(CUDA_INSTALL_PATH)/lib
endif

ifdef CUDA_INSTALL_PATH
	LDFLAGS += $(CUDA_LIB)
endif
LDFLAGS += -lcudart -lnpp

# Build
build: $(TARGET)

# Clean
clean:
	rm -f *.o $(TARGET)

# Lists of object files
COBJS=$(CFILES:.c=.c.o)
CUOBJS=$(CUFILES:.cu=.cu.o)

# Build target
ifeq ($(SHARED_LIBRARY),1)
    $(TARGET): $(COBJS) $(CUOBJS)
		$(LINK) $(COBJS) $(CUOBJS) $(LDFLAGS) -o $(TARGET);    
else
    $(TARGET): $(COBJS) $(CUOBJS)
    
		ar rcs $(TARGET) $(COBJS) $(CUOBJS);
endif

# Set suffix for CUDA files
.SUFFIXES: .cu

# Pattern rule for compiling C files
%.c.o: %.c 
	$(CC) $(CFLAGS) -c $< -o $@

# Pattern rule for compiling CUDA files
%.cu.o: %.cu
	$(NVCC) $(NVCCFLAGS) -c $< -o $@;

# Set file dependencies
gpujpeg_common.c.o: gpujpeg_common.c gpujpeg_common.h
gpujpeg_encoder.c.o: gpujpeg_encoder.c gpujpeg_encoder.h
gpujpeg_decoder.c.o: gpujpeg_decoder.c gpujpeg_decoder.h
gpujpeg_table.c.o: gpujpeg_table.c gpujpeg_table.h
gpujpeg_preprocessor.cu.o: gpujpeg_preprocessor.cu gpujpeg_preprocessor.h gpujpeg_colorspace.h
gpujpeg_dct_cpu.c.o: gpujpeg_dct_cpu.c gpujpeg_dct_cpu.h
gpujpeg_dct_gpu.cu.o: gpujpeg_dct_gpu.cu gpujpeg_dct_gpu.h
gpujpeg_huffman_cpu_encoder.c.o: gpujpeg_huffman_cpu_encoder.c gpujpeg_huffman_cpu_encoder.h
gpujpeg_huffman_gpu_encoder.cu.o: gpujpeg_huffman_gpu_encoder.cu gpujpeg_huffman_gpu_encoder.h
gpujpeg_huffman_cpu_decoder.c.o: gpujpeg_huffman_cpu_decoder.c gpujpeg_huffman_cpu_decoder.h
gpujpeg_huffman_gpu_decoder.cu.o: gpujpeg_huffman_gpu_decoder.cu gpujpeg_huffman_gpu_decoder.h
gpujpeg_writer.c.o: gpujpeg_writer.c gpujpeg_writer.h
gpujpeg_reader.c.o: gpujpeg_reader.c gpujpeg_reader.h
