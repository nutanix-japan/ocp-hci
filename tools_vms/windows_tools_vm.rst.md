---
title: Windows Tools VM
---

# Overview

This Windows Server 2012 R2 image comes pre-installed with a number of
tools, including:

-   Microsoft Remote Server Administration Tools (RSAT)
-   PuTTY, CyberDuck, WinSCP
-   Sublime Text 3, Visual Studio Code
-   OpenOffice
-   Python
-   pgAdmin
-   Chocolatey package manager
-   Google Chrome browser

Deploy this VM on your assigned cluster if directed to do so as part of
**Lab Setup**.

```{=html}
<body><font color="red">Only deploy the VM once, it does not need to be cleaned up as part of any lab completion.</font></body>
```
# Deploying Windows Tools VM

In **Prism Central** \> select `bars`{.interpreted-text role="fa"} **\>
Compute and Storage \> VMs**, and click **Create VM**.

1.  Fill out the following fields:
    -   **Name** - *Initials*-Windows-ToolsVM
    -   **Description** - (Optional) Description for your VM.
    -   **Number of VMs** - 1
    -   **vCPU(s)** - 1
    -   **Number of Cores per vCPU** - 2
    -   **Memory** - 2 GiB
2.  Click **Next**
3.  Under **Disks** select **Attach Disk**
    -   **Type** - DISK
    -   **Operation** - Clone from Image
    -   **Image** - WinToolsVM-Q1CY21.qcow2
    -   **Capacity** - leave at default size
    -   **Bus Type** - leave at default SCSI Setting
4.  Click **Save**
5.  Under **Networks** select **Attach to Subnet**
    -   **VLAN Name** - Primary
    -   **Network Connection State** - Connected
    -   **Assignment Type** - Assign with DHCP
6.  Click **Save**
7.  Click **Next** at the bottom
8.  In **Management** section
    -   **Categories** - leave blank
    -   **Timezone** - leave at default UTC
    -   **Guest Customization** - No customization
9.  Click **Create VM** at the bottom
10. Go back to **Prism Central** \> select `bars`{.interpreted-text
    role="fa"} **\> Compute and Storage \> VMs**
11. Select your *Initials*-Windows-ToolsVM
12. Under **Actions** drop-down menu, choose **Power On**

Login to the VM via RDP or Console session, using the following
credentials:

-   **Username** - NTNXLAB\\Administrator
-   **password** - nutanix/4u
