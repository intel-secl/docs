# Appendix

## PCR Definitions

### Red Had Enterprise Linux

#### TPM 2.0

<table>
<thead>
<tr class="header">
<th>PCR</th>
<th>Measurement Parameters</th>
<th>Description</th>
<th>Operating System</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>PCR 0</td>
<td><p>BIOS ROM and Flash Image</p>
<p>Initial Boot Block (Intel® BootGuard only)</p></td>
<td><p>This PCR is based solely on the BIOS version, and remains identical across all hosts using the same BIOS. This PCR is used as the PLATFORM Flavor.</p>
<p>(Intel® BootGuard only): Extends measurements based on the Intel® BootGuard profile configuration and production vs non-production ACM flags; ACM signature; BootGuard key manifest hash; Boot Policy Manifest Signature</p></td>
<td><ul>
<li><p>All</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 7</td>
<td>Intel® BootGuard configuration and profiles</td>
<td>Describes the success of the IBB measurement event.</td>
<td><ul>
<li><p>All (Intel® BootGuard only)</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 17</td>
<td>ACM</td>
<td>BIOS AC registration information<br />
Digest of Processor S-CRTM<br />
Digest of Policycontrol<br />
Digest of all matching elements used by the policy<br />
Digest of STM<br />
Digest of Capability field of OsSinitData<br />
Digest of MLE<br />
For TA hosts, this PCR includes measurements of the OS, InitRD, and UUID. This changes with every install due to InitRD and UUID change.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 18</td>
<td>MLE [Tboot +VMM]</td>
<td>Digest of public key modulus used to verify SINIT signature<br />
Digest of Processor S-CRTM<br />
Digest of Capability field of OSSinitData table<br />
Digest of PolicyControl field of used policy<br />
Digest of LCP</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 19</td>
<td><blockquote>
<p>OS Specific.</p>
</blockquote>
<ul>
<li><p>ESX and Trust Agent — non Kernel modules</p></li>
<li><p>Citrix Xen — OS</p></li>
<li><p>+ Init RD + UUID</p></li>
</ul></td>
<td><p>For ESXi and Trust Agent hosts, this PCR contains individual measurements of all of the non-Kernel modules.</p>
<p>For Linux hosts, this PCR is a measurement of the OS, InitRD, and UUID.</p></td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
</tbody>
</table>

### VMWare ESXi

#### TPM 1.2

<table>
<thead>
<tr class="header">
<th>PCR</th>
<th>Measurement Parameters</th>
<th>Description</th>
<th>Operating System</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>PCR 0</td>
<td>BIOS ROM and Flash Image</td>
<td>This PCR is based solely on the BIOS version, and remains identical across all hosts using the same BIOS. This PCR is used as the PLATFORM Flavor.</td>
<td><ul>
<li><p>All</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 17</td>
<td>ACM</td>
<td>This PCR measures the SINIT ACM, and is hardware platform-specific. This PCR is part of the PLATFORM Flavor.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 18</td>
<td>MLE [Tboot +VMM]</td>
<td>This PCR measures the tboot and hypervisor version. In ESXi hosts, only the tboot version is measured.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 19</td>
<td><blockquote>
<p>OS Specific.</p>
</blockquote>
<ul>
<li><p>ESX and Trust Agent — non Kernel modules</p></li>
<li><p>Citrix Xen — OS</p></li>
<li><p>+ Init RD + UUID</p></li>
</ul></td>
<td><p>For ESXi and Trust Agent hosts, this PCR contains individual measurements of all of the non-Kernel modules.</p>
<p>For Citrix Xen hosts, this PCR is a measurement of the OS, InitRD, and UUID.</p></td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 20</td>
<td><p>For ESXi only.</p>
<p>VM Kernel and VMK Boot</p></td>
<td>This PCR is used only by ESXi hosts and is blank for all other host types.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 22</td>
<td>Asset Tag</td>
<td>This PCR contains the measurement of the SHA1 of the Asset Tag Certificate provisioned to the TPM, if any.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li></li>
</ul></td>
</tr>
</tbody>
</table>

#### TPM 2.0

VMWare supports TPM 2.0 with Intel TXT starting in vSphere 6.7 Update 1.
Earlier versions will support TPM 1.2 only.

<table>
<thead>
<tr class="header">
<th>PCR</th>
<th>Measurement Parameters</th>
<th>Description</th>
<th>Operating System</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>PCR 0</td>
<td>BIOS ROM and Flash Image</td>
<td>This PCR is based solely on the BIOS version, and remains identical across all hosts using the same BIOS. This PCR is used as part of the PLATFORM flavor.</td>
<td><ul>
<li><p>All</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 17</td>
<td>ACM</td>
<td>This PCR measures the SINIT ACM, and is hardware platform-specific. This PCR is part of the PLATFORM Flavor.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 18</td>
<td>MLE [Tboot +VMM]</td>
<td>This PCR measures the tboot and hypervisor version. In ESXi hosts, only the tboot version is measured. This PCR is part of the PLATFORM Flavor.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 19</td>
<td><blockquote>
<p>OS Specific.</p>
</blockquote>
<ul>
<li><p>ESX and Trust Agent — non Kernel modules</p></li>
<li><p>Citrix Xen — OS</p></li>
<li><p>+ Init RD + UUID</p></li>
</ul></td>
<td>For ESXi this PCR contains individual measurements of all of the non-Kernel modules – this includes all of the VIBs installed on the ESXi host. This is part of the OS flavor. Note that two ESXi hosts with the same version of ESXi installed may require different OS flavors if different VIBs are installed.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li><p>Red Hat Enterprise Linux</p></li>
</ul></td>
</tr>
<tr class="odd">
<td>PCR 20</td>
<td><p>For ESXi only.</p>
<p>VM Kernel and VMK Boot</p></td>
<td>This PCR is used only by ESXi hosts for some host-specific measurements, and is part of the host-unique flavor.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
</ul></td>
</tr>
<tr class="even">
<td>PCR 22</td>
<td>Asset Tag</td>
<td>Asset Tag is not currently supported for TPM 2.0 with ESXi.</td>
<td><ul>
<li><p>VMware ESXi</p></li>
<li></li>
</ul></td>
</tr>
</tbody>
</table>


## Attestation Rules

<table>
<thead>
<tr class="header">
<th>Platform</th>
<th>TPM</th>
<th>Flavor Type</th>
<th>Rules to be verified</th>
<th>Comments</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>RHEL</td>
<td>2.0</td>
<td>HARDWARE</td>
<td><p>PcrMatchesConstant rule for PCR 0</p>
<p>PcrEventLogIncludes rule for PCR 17 (LCP_DETAILS_HASH, BIOSAC_REG_DATA, OSSINITDATA_CAP_HASH, STM_HASH, MLE_HASH, NV_INFO_HASH, tb_policy, CPU_SCRTM_STAT, HASH_START, LCP_CONTROL_HASH)</p>
<p>PcrEventLogIntegrity rule for PCR 17</p></td>
<td><p>Evaluation of PcrEventLogIncludes would not include initrd and vmlinuz modules. They would be handled in host_specific flavor.</p>
<p>Evaluation of PcrEventLogIntegrity rule would also include OS modules (initrd &amp; vmlinuz)</p></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>OS</td>
<td>PcrEventLogIntegrity rule for PCR 17</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>ASSET_TAG</td>
<td>AssetTagMatches rule</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>HOST_SPECIFIC</td>
<td>PcrEventLogIncludes rule for PCR 17 (initrd &amp; vmlinuz)</td>
<td></td>
</tr>
<tr class="odd">
<td>VMware ESXi</td>
<td>1.2</td>
<td>PLATFORM</td>
<td><p>PcrMatchesConstant rule for PCR 0</p>
<p>PcrMatchesConstant rule for PCR 17</p></td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>OS</td>
<td><p>PcrMatchesConstant rule for PCR 18</p>
<p>PcrMatchesConstant rule for PCR 20</p>
<p>PcrEventLogEqualsExcluding rule for PCR 19 (excludes dynamic modules based on component name)</p>
<p>PcrEventLogIntegrity rule for PCR 19</p></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>ASSET_TAG</td>
<td>PcrMatchesConstant rule for PCR 22</td>
<td></td>
</tr>
<tr class="even">
<td>VMware ESXi</td>
<td>2.0</td>
<td></td>
<td>NOT SUPPORTED</td>
<td></td>
</tr>
<tr class="odd">
<td>Windows</td>
<td>1.2</td>
<td>PLATFORM</td>
<td>PcrMatchesConstant rule for PCR 0</td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>OS</td>
<td><p>PcrMatchesConstant rule for PCR 13</p>
<p>PcrMatchesConstant rule for PCR 14</p></td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>ASSET_TAG</td>
<td>AssetTagMatches rule</td>
<td></td>
</tr>
<tr class="even">
<td>Windows</td>
<td>2.0</td>
<td>PLATFORM</td>
<td>PcrMatchesConstant rule for PCR 0</td>
<td></td>
</tr>
<tr class="odd">
<td></td>
<td></td>
<td>OS</td>
<td><p>PcrMatchesConstant rule for PCR 13</p>
<p>PcrMatchesConstant rule for PCR 14</p></td>
<td></td>
</tr>
<tr class="even">
<td></td>
<td></td>
<td>ASSET_TAG</td>
<td>AssetTagMatches rule</td>
<td>AssetTagMatches rule needs to be updated to verify the key-value pairs after verifying the tag certificate.</td>
</tr>
</tbody>
</table>
