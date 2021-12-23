## Hardware Considerations

Intel® SecL-DC supports and uses a variety of Intel security features, but there are some key requirements to consider before beginning an installation. Most important among these is the Root of Trust configuration. This involves deciding what combination of TXT, Boot Guard, tboot, and UEFI Secure Boot to enable on platforms that will be attested using Intel® SecL.

Key points:

\-   At least one "Static Root of Trust" mechanism must be used (TXT and/or BtG)

\-   For Legacy BIOS systems, tboot must be used (which requires TXT)

\-   For UEFI mode systems, UEFI SecureBoot must be used*

Use the chart below for a guide to acceptable configuration options. .

![hardware-considerations](./images/hardware_considerations.png)