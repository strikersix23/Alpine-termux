# Docker on Termux [in a VM]

Create a Linux VM and install Docker in it so you can (slowly) run x86 Docker containers on your Android device.

Recommended to use SSH or external keyboard to execute the following commands unless you want sore thumbs. See https://wiki.termux.com/wiki/Remote_Access#SSH

* Install QEMU
	```
	pkg install qemu-utils qemu-common qemu-system-x86_64-headless
	```

* Download Alpine Linux 3.17.2 (virt optimized) ISO
	```
	mkdir alpine && cd alpine
	wget https://dl-cdn.alpinelinux.org/alpine/v3.17/releases/x86_64/alpine-virt-3.17.2-x86_64.iso
	```
  

* Create disk (note it won't actually take 16GB of space, more like 500MB)
	```
	qemu-img create -f qcow2 alpine.img 16G
        ```
* Boot it up
  ```
  qemu-system-x86_64 -machine q35 -m 1024 -smp cpus=2 -cpu qemu64 \
    -drive if=pflash,format=raw,read-only,file=$PREFIX/share/qemu/edk2-x86_64-code.fd \
    -netdev user,id=n1,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 \
    -cdrom  alpine-virt-3.17.2-x86_64.iso \
    -nographic alpine.img
  ```

* Login with user `root` (no password)

<!-- aarch64 is too slow ```
qemu-system-aarch64 -machine virt -m 1024 -smp cpus=2 \
  -cpu cortex-a57 -bios QEMU_EFI.fd \
  -netdev user,id=n1 -device virtio-net,netdev=n1 \
  -cdrom alpine-virt-3.17.2-aarch64.iso \
  -nographic alpine.img
``` -->

* Setup network (press Enter to use defaults):
	```
	localhost:~# setup-interfaces
	Available interfaces are: eth0.
	Enter '?' for help on bridges, bonding and vlans.
	Which one do you want to initialize? (or '?' or 'done') [eth0] 
	Ip address for eth0? (or 'dhcp', 'none', '?') [dhcp] 
	Do you want to do any manual network configuration? [no] 
	localhost:~# ifup eth0
	```

* Create an answerfile to speed up installation:

  ```
  localhost:~# wget https://raw.githubusercontent.com/strikersix23/Alpine-termux/main/answerfile
  ```

* Patch `setup-disk` to enable serial console output on boot

  ```
  localhost:~# sed -i -E 's/(local kernel_opts)=.*/\1="console=ttyS0"/' /sbin/setup-disk
  ```

* Run setup to install to disk
  ```
  localhost:~# setup-alpine -f answerfile
  ```

* Once installation is complete, power off the VM (command `poweroff`) and boot again without cdrom:

  ```
  qemu-system-x86_64 -machine q35 -m 1024 -smp cpus=2 -cpu qemu64 \
    -drive if=pflash,format=raw,read-only,file=$PREFIX/share/qemu/edk2-x86_64-code.fd \
    -netdev user,id=n1,hostfwd=tcp::2222-:22 -device virtio-net,netdev=n1 \
    -nographic alpine.img
  ```

* Install docker and enable on boot:
  ```
  alpine:~# apk update && apk add docker
  alpine:~# service docker start
  alpine:~# rc-update add docker
  ```

* Useful keys:
  * Ctrl+a x: quit emulation
  * Ctrl+a h: toggle QEMU consolee an answerfile to speed up installation:
