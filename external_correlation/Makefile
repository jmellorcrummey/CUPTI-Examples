#
# Copyright 2011-2015 NVIDIA Corporation. All rights reserved
# 
CUDA=/usr/local/cuda-9.0
CUPTI=$(CUDA)/extras/CUPTI
INCLUDES=-I$(CUDA)/include -I$(CUPTI)/include
CUPTI_LIBDIR=$(CUPTI)/lib64

ifndef OS
 OS   := $(shell uname)
 HOST_ARCH := $(shell uname -m)
endif

FLAGS= -Xcompiler "-fopenmp -g" -std=c++11 -G

ARCH= -arch sm_52

ifeq ($(OS),Windows_NT)
    export PATH := $(PATH):../../libWin32:../../libx64
    LIBS= -lcuda -L ../../libWin32 -L ../../libx64 -lcupti
    OBJ = obj
else
    ifeq ($(OS), Darwin)
        export DYLD_LIBRARY_PATH := $(DYLD_LIBRARY_PATH):../../lib
        LIBS= -Xlinker -framework -Xlinker cuda -L ../../lib -lcupti
    else
        export LD_LIBRARY_PATH := $(LD_LIBRARY_PATH):../../lib:../../lib64
        LIBS= -lcuda -L $(CUPTI_LIBDIR) -Xlinker -rpath=$(CUPTI_LIBDIR) -lcupti
    endif
    OBJ = o
endif

CUBIN = cubin

all: vec external_id.$(OBJ) vec.$(OBJ) vecSub.$(CUBIN) vecAdd.$(CUBIN)

vec: external_id.$(OBJ) vec.$(OBJ)
	nvcc -o $@ $^ $(FLAGS) $(ARCH) $(LIBS)

external_id.$(OBJ): external_id.cpp
	nvcc -c $(INCLUDES) $(FLAGS) $(ARCH) $<

vec.$(OBJ): vec.cu
	nvcc -c $(INCLUDES) $(FLAGS) $(ARCH) $<

vecSub.$(CUBIN): vecSub.cu
	nvcc -cubin $(INCLUDES) $(FLAGS) $(ARCH) $<

vecAdd.$(CUBIN): vecAdd.cu
	nvcc -cubin $(INCLUDES) $(FLAGS) $(ARCH) $<

run: vec
	./$<

test-runtime:
	echo test: normal kernel launch does not report external correlation ids in activity API trace records
	./vec runtime

test-same-context:
	echo test: two threads sharing a cuda context
	./vec same_context

test-diff-context:
	echo test: two threads each with their own cuda context
	./vec diff_context

test-diff-context:
	echo test: two threads each with their own cuda context with each thread deleting its context
	./vec diff_context_delete

test-all: test-runtime test-same-context test-diff-context test-diff-context-delete

clean:
	rm -f vec external_id.$(OBJ) vec.$(OBJ) vecSub.$(CUBIN) vecAdd.$(CUBIN)
