# nixos-conf

NixOS flake configurations

## Getting Started 

### NixOS Installer

There are two hardware choices to get booted into a NixOS installer:
1. Physical
2. VM

#### Physical

TODO

#### VM

1. Create a VM
```
$ quickget nixos 24.11 minimal
```

2. Start the VM

```
$ quickemu --vm nixos-24.11-minimal.conf
~/nixos ~/nixos
Quickemu 4.9.5 using /nix/store/sqcqpkqvbxsx8wmc6bixdgfxjmibxmdw-qemu-8.2.7/bin/qemu-system-x86_64 v8.2.7
 - Host:     NixOS 24.05 (Uakari) running Linux 6.12.1 i7dwarf
 - CPU:      12th Gen Intel(R) Core(TM) i7-12700H 
 - CPU VM:   host, 1 Socket(s), 4 Core(s), 2 Thread(s)
 - RAM VM:   8G RAM
 - BOOT:     EFI (Linux), OVMF (/nix/store/q5qb5j8c03ywaak115a51wk9551jralb-OVMF-202402-fd/FV/OVMF_CODE.fd), SecureBoot (off).
 - Disk:     nixos-24.11-minimal/disk.qcow2 (32G)
             Just created, booting from nixos-24.11-minimal/latest-nixos-minimal-x86_64-linux.iso
 - Boot ISO: nixos-24.11-minimal/latest-nixos-minimal-x86_64-linux.iso
 - Display:  SDL, virtio-vga-gl, GL (on), VirGL (on) @ (1280 x 800)
 - Sound:    intel-hda (hda-micro)
 - ssh:      On host:  ssh user@localhost -p 22220
 - WebDAV:   On guest: dav://localhost:9843/
 - 9P:       On guest: sudo mount -t 9p -o trans=virtio,version=9p2000.L,msize=104857600 Public-cat ~/Public
 - smbd:     On guest: smb://10.0.2.4/qemu
 - Network:  User (virtio-net)
 - Monitor:  On host:  nc -U "nixos-24.11-minimal/nixos-24.11-minimal-monitor.socket"
             or     :  socat -,echo=0,icanon=0 unix-connect:nixos-24.11-minimal/nixos-24.11-minimal-monitor.socket
 - Serial:   On host:  nc -U "nixos-24.11-minimal/nixos-24.11-minimal-serial.socket"
             or     :  socat -,echo=0,icanon=0 unix-connect:nixos-24.11-minimal/nixos-24.11-minimal-serial.socket
 - Process:  Started nixos-24.11-minimal.conf as nixos-24.11-minimal (3753507)
```

3. Set the password for the `nixos` user within the VM terminal.

```
[nixos@nixos:~]$ passwd
```

4. ssh into VM

```
$ ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no nixos@localhost -p 22220
[nixos@nixos:~]$
```

### Initial Configuration

1. Generate the nixos hardware config

```
[nixos@nixos:~]$ nixos-generate-config --dir /tmp
writing /tmp/hardware-configuration.nix...
writing /tmp/configuration.nix...
For more hardware-specific settings, see https://github.com/NixOS/nixos-hardware.
```

2. Copy the boot settings to `./nixos/<machine-name>/default.nix`. Ex: copy the fields that contain non-empty values which are the availableKernelModules and kernelModules.

```
[nixos@nixos:~]$ mkdir -p ./nixos/proof
[nixos@nixos:~]$ grep -i boot.*module /tmp/hardware-configuration.nix | tee ./nixos/proof/default.nix
  boot.initrd.availableKernelModules = [ "xhci_pci" "ohci_pci" "ehci_pci" "virtio_pci" "virtio_scsi" "ahci" "usbhid" "sd_mod" "sr_mod" ];
  boot.initrd.kernelModules = [ ];
  boot.kernelModules = [ "kvm-intel" ];
  boot.extraModulePackages = [ ];
```

3. Download wimpy's install script and modify it. TODO fork it.

```
[nixos@nixos:~]$ curl -sL https://raw.githubusercontent.com/wimpysworld/nix-config/main/nixos/_mixins/scripts/install-system/./install-system.sh -O install-system.sh
[nixos@nixos:~]$ chmod u+x install-system.sh 
[nixos@nixos:~]$ sed -i 's,martin,cat,g' install-system.sh 
[nixos@nixos:~]$ sed -i 's,wimpysworld/nix-config,tracemeyers/nixos-conf,g' install-system.sh 
```
