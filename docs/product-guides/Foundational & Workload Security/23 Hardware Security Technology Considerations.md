# Hardware Security Technology Considerations

Intel® SecL-DC supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

Key points:

\-   At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG)

\-   For Legacy BIOS systems, tboot must be used (which requires TXT)

\-   For UEFI mode systems, UEFI SecureBoot must be used

---
**NOTE**
Currently tboot and Secure Boot are not compatible. For UEFI platforms, Intel reccomends enabling TXT and enabling Secure Boot.  If Secure Boot will not be used, then Intel recommends enabling TXT and installing tboot.

These hardware security technology requirements apply to all platforms to be attested.  In a Kubernetes environment, this would typically include all worker nodes.
---

Use the chart below for a guide to acceptable configuration options. .

![hardware-considerations](./images/hardware_considerations.png)



# Tboot Installation

Tboot is required to build a complete Chain of Trust for Intel® TXT systems that are not using UEFI Secure Boot. Tboot acts to initiate the Intel® TXT SINIT ACM (Authenticated Code Module), which populates several TPM measurements including measurement of the kernel, grub command line, and initrd. Without either tboot or UEFI Secure Boot, the Chain of Trust will be broken because the OS-related components will be neither measured nor signature-verified prior to execution. Because tboot acts to initiate the Intel® TXT SINIT ACM, tboot is only required for platforms using Intel® TXT, and is not required for platforms using another hardware Root of Trust technology like Intel® Boot Guard.

---
**Note:**
SGX Attestation fails when SGX is enabled on a host booted using tboot.

Root Cause: tboot requires the "noefi" kernel parameter to be passed during boot, in order to not use unmeasured EFI runtime services. As a result, the kernel does not expose EFI variables to user-space. SGX Attestation requires these EFI variables to fetch Platform Manifest data.

**Workaround:**

The EFI variables required by SGX are only needed during the SGX provisioning/registration phase. Once this step is completed successfully, access to the EFI variables is no longer required. This means this issue can be worked around by installing the SGX agent without booting to tboot, then rebooting the system to tboot. SGX attestation will then work as expected while booted to tboot.

- Enable SGX and TXT in the platform BIOS

- Perform SGX Factory Reset and boot into the “plain” distribution kernel (without tboot or TCB)

- Install tboot and ISecL components (SGX Agent, Trust Agent and Workload Agent)

The SGX Agent installation fetches the SGX Platform Manifest data and caches it

- Reboot the system into the tboot kernel mode.

- Verify that TXT measured launch was successful:

```
​ txt-stat |grep "TXT measured launch"
```

The SGX and Platform Integrity Attestation use cases should now work as normal.

---


Intel® SecL-DC requires tboot 1.10.1 or greater. This may be a later version of tboot than is available on public software repositories.

The most current version of tboot can be found here:

https://sourceforge.net/projects/tboot/files/tboot/

Tboot requires configuration of the grub boot loader after installation. To install and configure tboot:

## Install tboot

```
yum install tboot
```

If the package manager does not support a late enough version of tboot, it will need to be compiled from source and installed manually. Instructions can be found here:

https://sourceforge.net/p/tboot/wiki/Home/

---
**NOTE**
The step "copy platform SINIT to /boot" should not be required, as datacenter platforms include the SINIT in the system BIOS package.

---

## Ensure that multiboot2.mod and relocator.mod are available for grub2

This step may not be necessary for all OS versions. For instance, this step is not applicable for tboot installation on Ubuntu 18.04. In order to utilize tboot, grub2 requires these two modules from the grub2-efi-x64-modules package to be located in the correct directory (if they're absent, the host will throw a grub error when it tries to boot using tboot).

These files must be present in this directory:

```
/boot/efi/EFI/redhat/x86_64-efi/multiboot2.mod
/boot/efi/EFI/redhat/x86_64-efi/relocator.mod
```

If the files are not present in this directory, they can be moved from their installation location:

```
cp /usr/lib/grub/x86_64-efi/multiboot2.mod /boot/efi/EFI/redhat/x86_64-efi/
cp /usr/lib/grub/x86_64-efi/relocator.mod /boot/efi/EFI/redhat/x86_64-efi/
```

Make a backup of your current grub.cfg file

The below examples assume a RedHat OS that has been installed on a platform using UEFI boot mode. The grub path will be slightly different for platforms using a non-RedHat OS.

```
cp /boot/efi/EFI/redhat/grub.cfg /boot/efi/EFI/redhat/grub.cfg.bak
```

Generate a new grub.cfg with the tboot boot option

For RHEL:

```
grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

For Ubuntu:

```
grub-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
Update the default boot option
```

## Ensure that the GRUB_DEFAULT value is set to the tboot option.

Update /etc/default/grub and set the GRUB_DEFAULT value to 'tboot-1.10.1'

```
GRUB_DEFAULT='tboot-1.10.1'
```

## Regenerate grub.cfg:

For RHEL:

```
grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

For Ubuntu:

```
grub-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
```

Reboot the system.  Because measurement happens at system boot, a reboot is needed to boot to the tboot boot option and populate measurements in the TPM.

## Verify a successful trusted boot with tboot

Tboot provides the txt-stat command to show the tboot log. The first part of the output of this command can be used to verify a successful trusted launch. In the output below, note the “TXT measured launch” and “secrets flag set” at the bottom. Both of these should show "TRUE" if the tboot measured launch was successful. If either of these show "FALSE" the measured launch has failed. This usually simply indicates that the tboot boot option was not selected during boot.

```
txt-stat
```

If the measured launch was successful, proceed to install the Trust Agent.

```
Intel(r) TXT Configuration Registers:
        STS: 0x0001c091
            senter_done: TRUE
            sexit_done: FALSE
            mem_config_lock: FALSE
            private_open: TRUE
            locality_1_open: TRUE
            locality_2_open: TRUE
        ESTS: 0x00
            txt_reset: FALSE
        E2STS: 0x0000000000000006
            secrets: TRUE
        ERRORCODE: 0x00000000
        DIDVID: 0x00000001b0078086
            vendor_id: 0x8086
            device_id: 0xb007
            revision_id: 0x1
        FSBIF: 0xffffffffffffffff
        QPIIF: 0x000000009d003000
        SINIT.BASE: 0x6fec0000
        SINIT.SIZE: 262144B (0x40000)
        HEAP.BASE: 0x6ff00000
        HEAP.SIZE: 1048576B (0x100000)
        DPR: 0x0000000070000051
            lock: TRUE
            top: 0x70000000
            size: 5MB (5242880B)
        PUBLIC.KEY:
            9c 78 f0 d8 53 de 85 4a 2f 47 76 1c 72 b8 6a 11
            16 4a 66 a9 84 c1 aa d7 92 e3 14 4f b7 1c 2d 11

***********************************************************
         **TXT measured launch: TRUE**
         **secrets flag set: TRUE**
***********************************************************
```