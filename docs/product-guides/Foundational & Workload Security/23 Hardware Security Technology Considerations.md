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