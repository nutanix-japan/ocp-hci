.. _windows_tools_vm:

----------------
Windows Tools VM
----------------

Overview
+++++++++

This Windows Server 2012 R2 image comes pre-installed with a number of tools, including:

- Microsoft Remote Server Administration Tools (RSAT)
- PuTTY, CyberDuck, WinSCP
- Sublime Text 3, Visual Studio Code
- OpenOffice
- Python
- pgAdmin
- Chocolatey Package Manager

Deploy this VM on your assigned cluster if directed to do so as part of **Lab Setup**.

.. raw:: html

  <strong><font color="red">Only deploy the VM once, it does not need to be cleaned up as part of any lab completion.</font></strong>

Deploying Windows Tools VM
++++++++++++++++++++++++++


In **Prism Central** > select :fa:`bars` **> Compute and Storage > VMs**, and click **Create VM**.

#. Fill out the following fields:

   - **Name** - *Initials*-Windows-ToolsVM
   - **Description** - (Optional) Description for your VM.
   - **Number of VMs** - 1
   - **vCPU(s)** - 1
   - **Number of Cores per vCPU** - 2
   - **Memory** - 2 GiB

#. Click **Next**

#. Under **Disks** select **Attach Disk**

   - **Type** - DISK
   - **Operation** - Clone from Image 
   - **Image** - Linux_ToolsVM.qcow2
   - **Capacity** - leave at default size
   - **Bus Type** - leave at default SCSI Setting

#. Click **Save**

#. Under **Networks** select **Attach to Subnet**

   - **VLAN Name** - Primary
   - **Network Connection State** - Connected
   - **Assignment Type** - Assign with DHCP

#. Click **Save**

#. Click **Next** at the bottom

#. In Management page 
   
   - **Categories** - leave blank
   - **Timezone** - leave at default UTC
   - **Guest Customization** - No customization 

#. Click **Create VM** at the bottom

#. Go back to **Prism Central** > select :fa:`bars` **> Compute and Storage > VMs**

#. Select your *Initials*-Windows-ToolsVM

#. Under **Actions** drop-down menu, choose **Power On**

Login to the VM via RDP or Console session, using the following credentials:

- **Username** - NTNXLAB\\Administrator
- **password** - nutanix/4u
