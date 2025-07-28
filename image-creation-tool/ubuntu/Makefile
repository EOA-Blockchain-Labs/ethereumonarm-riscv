DATE := $(shell date +"%y.%m")
RELEASE := 00

define download_image
	mkdir -p /tmp/ubuntu_iso
	wget $1
	unxz $(notdir $1)
endef

define modify_image
	sudo mount -o loop,offset=$(2) $(notdir $1) /tmp/ubuntu_iso
	sudo cp -a sources/etc/rc.local.$(3) /tmp/ubuntu_iso/etc/rc.local
	sudo sed -i 's/1/0/' /tmp/ubuntu_iso/etc/apt/apt.conf.d/20auto-upgrades
	sync
	sudo umount /tmp/ubuntu_iso
	mv $(notdir $1) ethonarm_$(3)_$(DATE).$(RELEASE).img
	rmdir /tmp/ubuntu_iso
endef

visionfive2_url := https://cdimage.ubuntu.com/releases/23.04/release/ubuntu-23.04-preinstalled-server-riscv64+visionfive2.img.xz 
visionfive2_iso := ubuntu-23.04-preinstalled-server-riscv64+visionfive2.img
visionfive2_offset := 269484032
visionfive2_rclocal := visionfive2

all: visionfive2

visionfive2:
	$(call download_image,$(visionfive2_url))
	$(call modify_image,$(visionfive2_iso),$(visionfive2_offset),$(visionfive2_rclocal))

