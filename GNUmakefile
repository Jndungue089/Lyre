QEMUFLAGS ?= -M q35,smm=off -m 8G -boot order=dc -cdrom lyre.iso -serial stdio -smp 4

.PHONY: all
all:
	rm -f lyre.iso
	$(MAKE) lyre.iso

lyre.iso: jinx
	rm -f builds/kernel.built builds/kernel.packaged
	$(MAKE) distro-base
	./build-support/makeiso.sh

.PHONY: debug
debug:
	JINX_CONFIG_FILE=jinx-config-debug $(MAKE) all

jinx:
	curl -Lo jinx https://github.com/mintsuki/jinx/raw/b06cbf4cf142ff0f3aa0ab8438ede29951887b43/jinx
	chmod +x jinx

.PHONY: distro-full
distro-full: jinx
	./jinx build-all

.PHONY: distro-base
distro-base: jinx
	./jinx build base-files kernel init bash coreutils nano less binutils gcc

.PHONY: run-kvm
run-kvm: lyre.iso
	qemu-system-x86_64 -enable-kvm -cpu host $(QEMUFLAGS)

.PHONY: run-hvf
run-hvf: lyre.iso
	qemu-system-x86_64 -accel hvf -cpu host $(QEMUFLAGS)

ovmf:
	mkdir -p ovmf
	cd ovmf && curl -o OVMF.fd https://retrage.github.io/edk2-nightly/bin/RELEASEX64_OVMF.fd

.PHONY: run-uefi
run-uefi: lyre.iso ovmf
	qemu-system-x86_64 -enable-kvm -cpu host $(QEMUFLAGS) -bios ovmf/OVMF.fd

.PHONY: run
run: lyre.iso
	qemu-system-x86_64 $(QEMUFLAGS)

.PHONY: kernel-clean
kernel-clean:
	rm -rf builds/kernel* pkgs/kernel*

.PHONY: init-clean
init-clean:
	rm -rf builds/init* pkgs/init*

.PHONY: base-files-clean
base-files-clean:
	rm -rf builds/base-files* pkgs/base-files*

.PHONY: clean
clean: kernel-clean init-clean base-files-clean
	rm -rf iso_root sysroot lyre.iso initramfs.tar

.PHONY: distclean
distclean: jinx
	cd kernel && ./bootstrap && ./configure && make maintainer-clean
	./jinx clean
	rm -rf iso_root sysroot lyre.iso initramfs.tar jinx ovmf
	chmod -R 777 .jinx-cache
	rm -rf .jinx-cache
