## Appendix

**Important Note:** SGX Attestation fails when SGX is enabled on a host booted using tboot

**Root Cause:** tboot requires the "noefi" kernel parameter to be passed during boot, in order to not use an unmeasured EFI runtime services. As a result, the kernel does not expose EFI variables to user-space. SGX Attestation requires these EFI variables to fetch Platform Manifest data. 

**Workaround:**

The EFI variables required by SGX are only needed during the SGX provisioning/registration phase. Once this step is completed successfully, access to the EFI variables is no longer required. This means this issue can be worked around by installing the SGX agent without booting to tboot, then rebooting the system to tboot. SGX attestation will then work as expected while booted to tboot.

1. Enable SGX and TXT in the platform BIOS

2. Perform SGX Factory Reset and boot into the “plain” distribution kernel (without tboot or TCB)

3. Install tboot and ISecL components (SGX Agent, Trust Agent and Workload Agent)

4. The SGX Agent installation fetches the SGX Platform Manifest data and caches it

5. Reboot the system into the tboot kernel mode.

6. Verify that TXT measured launch was successful:

   `txt-stat |grep "TXT measured launch"`

7. The SGX and Platform Integrity Attestation use cases should now work as normal.
