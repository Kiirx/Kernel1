.PHONY: all
all: kernel1.iso

.PHONY: all-hdd
all-hdd: kernel1.hdd

.PHONY: run
run: kernel1.iso
	qemu-system-x86_64 -M q35 -m 1G -smp 8 -serial stdio -cdrom kernel1.iso -boot d

.PHONY: run-uefi
run-uefi: ovmf-x64 kernel1.iso
	qemu-system-x86_64 -M q35 -m 8G -smp 8 -bios ovmf-x64/OVMF.fd -cdrom kernel1.iso -boot d

.PHONY: run-hdd
run-hdd: kernel1.hdd
	qemu-system-x86_64 -M q35 -m 8G -smp 8 -hda kernel1.hdd

.PHONY: run-hdd-uefi
run-hdd-uefi: ovmf-x64 kernel1.hdd
	qemu-system-x86_64 -M q35 -m 8G -smp 8 -bios ovmf-x64/OVMF.fd -hda kernel1.hdd

ovmf-x64:
	mkdir -p ovmf-x64
	cd ovmf-x64 && curl -o OVMF-X64.zip https://efi.akeo.ie/OVMF/OVMF-X64.zip && 7z x OVMF-X64.zip

limine:
	git clone https://github.com/limine-bootloader/limine.git --branch=v3.0-branch-binary --depth=1
	make -C limine

.PHONY: kernel
kernel:
	$(MAKE) -C kernel

kernel1.iso: limine kernel
	rm -rf iso_root
	mkdir -p iso_root
	cp kernel/kernel.elf modules/background.bmp \
		limine.cfg limine/limine.sys limine/limine-cd.bin limine/limine-cd-efi.bin iso_root/
	xorriso -as mkisofs -b limine-cd.bin \
		-no-emul-boot -boot-load-size 4 -boot-info-table \
		--efi-boot limine-cd-efi.bin \
		-efi-boot-part --efi-boot-image --protective-msdos-label \
		iso_root -o kernel1.iso
	limine/limine-deploy kernel1.iso
	rm -rf iso_root

kernel1.hdd: limine kernel
	rm -f kernel1.hdd
	dd if=/dev/zero bs=1M count=0 seek=64 of=kernel1.hdd
	parted -s kernel1.hdd mklabel gpt
	parted -s kernel1.hdd mkpart ESP fat32 2048s 100%
	parted -s kernel1.hdd set 1 esp on
	limine/limine-deploy kernel1.hdd
	sudo losetup -Pf --show kernel1.hdd >loopback_dev
	sudo mkfs.fat -F 32 `cat loopback_dev`p1
	mkdir -p img_mount
	sudo mount `cat loopback_dev`p1 img_mount
	sudo mkdir -p img_mount/EFI/BOOT
	sudo cp -v kernel/kernel.elf limine.cfg limine/limine.sys img_mount/
	sudo cp -v limine/BOOTX64.EFI img_mount/EFI/BOOT/
	sync
	sudo umount img_mount
	sudo losetup -d `cat loopback_dev`
	rm -rf loopback_dev img_mount

.PHONY: clean
clean:
	rm -rf iso_root kernel1.iso kernel1.hdd
	$(MAKE) -C kernel clean

.PHONY: distclean
distclean: clean
	rm -rf limine ovmf-x64
	$(MAKE) -C kernel distclean
