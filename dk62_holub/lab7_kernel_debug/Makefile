# (!) using paths with spaces may not work with Kbuild
INCDIR := inc
SRCDIR := src
obj-m := hivemod.o

# this is what is used by Kbuild
obj-m += $(addprefix $(SRCDIR)/,$(OBJ))

EXTRA_CFLAGS += -I$(INCDIR) -std=gnu11

# directory containing Makefile for kernel build
KBUILDDIR := /lib/modules/$(shell uname -r)/build/

.PHONY: modules clean tidy all 

all: | tidy modules test

.build:
	cp -r ./$(SRCDIR) ./$@
	cp Makefile $@/

# recur to the original kernel Makefile with some additions
modules: .build
	cd $< && $(MAKE) -C "$(KBUILDDIR)" M="$(PWD)/$<" modules

tidy:
	-rm -rf .build
test:
	cd .build
	gcc ../src/test.c -o test
	sudo sh -c "insmod $(obj-m:.o=.ko); sleep 1; ./test; rmmod $(obj-m:.o=); sleep 1; dmesg -k | tail -10"
	

# we link original clean to target named tidy
.ONESHELL:
clean:
	cd .build
	-rm -f modules.order Module.symvers
	-rm -rf .tmp_versions
	-rm test
	-rm -f $(obj-m) $(obj-m:.o=.mod.o) $(obj-m:.o=.mod.c)
	-rm -f $(addsuffix .cmd,$(addprefix .,$(obj-m)))
	-rm -f $(addsuffix .cmd,$(addprefix .,$(obj-m:.o=.ko)))
	-rm -f $(addsuffix .cmd,$(addprefix .,$(obj-m:.o=.mod.o)))

