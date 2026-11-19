#RISCV=riscv64-unknown-elf-
RISCV=~/riscv-entropy_injection/install/compiler/bin/riscv32-unknown-elf-
RISCV_GCC_OPTS=-march=rv32i -mabi=ilp32 -mstrict-align -nostartfiles common/crt.S

all: strcpy

.PHONY: test
test: test/test.c
	$(RISCV)gcc $(RISCV_GCC_OPTS) $< -o $@/$@.out
	$(RISCV)objdump -d $@/$@.out > $@/$@.out.dmp
	$(RISCV)objcopy -O binary $@/$@.out $@/$@.bin
	hexdump $@/$@.bin > $@/$@.bin.dmp

.PHONY: atoi
atoi: atoi/atoi.c
	$(RISCV)gcc $(RISCV_GCC_OPTS) $< -o $@/$@.out
	$(RISCV)objdump -d $@/$@.out > $@/$@.out.dmp
	$(RISCV)objcopy -O binary $@/$@.out $@/$@.bin
	hexdump $@/$@.bin > $@/$@.bin.dmp

.PHONY: memset
memset: memset/memset.c
	$(RISCV)gcc $(RISCV_GCC_OPTS) $< -o $@/$@.out
	$(RISCV)objdump -d $@/$@.out > $@/$@.out.dmp
	$(RISCV)objcopy -O binary $@/$@.out $@/$@.bin
	hexdump $@/$@.bin > $@/$@.bin.dmp

.PHONY: itoa
itoa: itoa/itoa.c
	$(RISCV)gcc $(RISCV_GCC_OPTS) $< -o $@/$@.out
	$(RISCV)objdump -d $@/$@.out > $@/$@.out.dmp
	$(RISCV)objcopy -O binary $@/$@.out $@/$@.bin
	hexdump $@/$@.bin > $@/$@.bin.dmp

.PHONY: strcpy
strcpy: strcpy/strcpy.c
	$(RISCV)gcc $(RISCV_GCC_OPTS) $< -o $@/$@.out
	$(RISCV)objdump -d $@/$@.out > $@/$@.out.dmp
	$(RISCV)objcopy -O binary $@/$@.out $@/$@.bin
	hexdump $@/$@.bin > $@/$@.bin.dmp

.PHONY: loop_acc
loop_acc: loop_acc/loop_acc.S
	$(RISCV)gcc $(RISCV_GCC_OPTS) $< -o $@/$@.out
	$(RISCV)objdump -d $@/$@.out > $@/$@.out.dmp
	$(RISCV)objcopy -O binary $@/$@.out $@/$@.bin
	hexdump $@/$@.bin > $@/$@.bin.dmp

.PHONY: fibonacci
fibonacci: fibonacci/fibonacci.c
	$(RISCV)gcc $(RISCV_GCC_OPTS) $< -o $@/$@.out
	$(RISCV)objdump -d $@/$@.out > $@/$@.out.dmp
	$(RISCV)objcopy -O binary $@/$@.out $@/$@.bin
	hexdump $@/$@.bin > $@/$@.bin.dmp

.PHONY: endless_loop
endless_loop: endless_loop/endless_loop.S
	$(RISCV)gcc $(RISCV_GCC_OPTS) $< -o $@/$@.out
	$(RISCV)objdump -d $@/$@.out > $@/$@.out.dmp
	$(RISCV)objcopy -O binary $@/$@.out $@/$@.bin
	hexdump $@/$@.bin > $@/$@.bin.dmp

.PHONY: clean
clean:
	rm -rf */*.bin */*.out */*.dmp
