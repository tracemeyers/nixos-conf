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

2. Download wimpy's install script and modify it with your repo and default user. TODO fork it.

```
[nixos@nixos:~]$ curl -sL https://raw.githubusercontent.com/wimpysworld/nix-config/main/nixos/_mixins/scripts/install-system/./install-system.sh -O install-system.sh
[nixos@nixos:~]$ chmod u+x install-system.sh
[nixos@nixos:~]$ sed -i 's,wimpysworld/nix-config,tracemeyers/nixos-conf,g' install-system.sh 
[nixos@nixos:~]$ sed -i 's,martin,cat,g' install-system.sh 
```

3. Run the installer so it sets up the git repo and then FAIL.

```
[nixos@nixos:~]$ ./install-system.sh proof cat
umount: /mnt: not mounted
Cloning into '/home/nixos/Zero/nix-config'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 12 (delta 2), reused 0 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (12/12), 4.98 KiB | 4.98 MiB/s, done.
Resolving deltas: 100% (2/2), done.
~/Zero/nix-config ~
Already on 'main'
Your branch is up to date with 'origin/main'.
WARNING! /home/nixos/.config/sops/age/keys.txt was not found.
         Do you want to continue without it?

Are you sure? [y/N]y
ERROR! install-system.sh could not find the required nixos/proof/disks.nix
```

4. Copy `hardware-configuration.nix` to `~/Zero/nix-config/nixos/<machine>/default.nix`

```
[nixos@nixos:~/Zero/nix-config]$ mkdir -p ~/Zero/nix-config/nixos/proof
[nixos@nixos:~/Zero/nix-config]$ cp /tmp/hardware-configuration.nix ~/Zero/nix-config/nixos/proof/default.nix
```

5. Modify `~/Zero/nix-config/nixos/<machine>/default.nix` to...
- Remove everything except the first line, imports, and boot fields.
- Add to imports `disks.nix`

```
{ modulesPath, ... }:

{
  imports =
    [ (modulesPath + "/profiles/qemu-guest.nix")
      ./disks.nix
    ];

  boot.initrd.availableKernelModules = [ "xhci_pci" "ohci_pci" "ehci_pci" "virtio_pci" "ahci" "usbhid" "sr_mod" "virtio_blk" ];
  boot.initrd.kernelModules = [ ];
  boot.kernelModules = [ "kvm-intel" ];
  boot.extraModulePackages = [ ];
}
```

6. Add `~/Zero/nix-config/nixos/<machine>/disks.nix`. Customize it as needed.

```
{ disks ? [ "/dev/vda" ], ... }:
{
  disko.devices = {
    disk = {
      vda = {
        type = "disk";
        device = builtins.elemAt disks 0;
        content = {
          type = "table";
          format = "gpt";
          partitions = [{
            name = "ESP";
            start = "0%";
            end = "550MiB";
            bootable = true;
            flags = [ "esp" ];
            fs-type = "fat32";
            content = {
              type = "filesystem";
              format = "vfat";
              mountpoint = "/boot";
            };
          }
          {
            name = "root";
            start = "550MiB";
            end = "100%";
            content = {
              type = "filesystem";
              extraArgs = [ ];
              format = "ext4";
              mountpoint = "/";
            };
          }];
        };
      };
    };
  };
}
```
