.. _ocp_assisted_install:

---------------------------
OCP: Assisted Installation 
---------------------------

In this optional lab we will go through Assited Installation feature that Red Hat Console offers.

Assisted Installer is a GUI driven way of deploying OCP clusters:

Pre-requisites for Assisted Installation
+++++++++++++++++++++++++++++++++++++++++

- Administrator creates the OCP Master and Worker VMs
- Adminsitrator provisions compute, networking and storage associated with the OCP Master and Worker VMs
- Access to Red Hat Console to use Assisted Installer and install a OCP cluster

High Level Overview of Assisted Installation
++++++++++++++++++++++++++++++++++++++++++++

Assisted Installer does the following:

- Provides RHOCS and OCP installation binaries in a CD-ROM ISO file
- Once the OCP VMs (Master and Worker) nodes are booted with this CD-ROM ISO file, using the public key the VMs connect to Red Hat Console
- VMs show in Assisted Installer page and the administrator (you) begins the installation
- Red Hat Console will manage and monitor the installation process from start to end 
- Upon successfull installatino of a OCP cluster the following will be provided:
  
  - KUBECONFIG file for ``oc`` access
  - DNS entries for OCP Cluster Manager access 

- Assisted installer will choose one of the Master VMs to serve the Bootstrap role 

.. note::

  Assisted Installers usage is in `Technical Preview <https://access.redhat.com/support/offerings/techpreview>`_ stage with Red Hat. 
  
  The OCP clusters deployed using Assisted Installers can be used for testing and development purposes.

At a very high level, we will accomplish the following to get a OCP cluster deployed using Assisted Installer:

1. Provision OCP Cluster in Red Hat Console and Generate CD-ROM ISO URL 
3. Provision OCP Infrastructure - Create Master and Worker VMs in your AHV HPOC/SPOC cluster using Terraform infrastructure as code
4. Install OCP Cluster in Red Hat Console

Provision OCP Cluster in Red Hat Console
+++++++++++++++++++++++++++++++++++++++++

#. Go to `Red Hat Console <https://console.redhat.com/openshift/assisted-installer/clusters>`_

#. Click on **Create new assisted cluster**

#. Fill in the following details:
   
   - **Cluster name** - Initials-assisted-cluster (e.g. xyz-assisted-cluster)
   - **Base domain** - ntnxlab.local

#. Click on **Next**

#. Click on **Add Host**

#. In the **Add Host** pop-up window select **Minimal image file: Provision with virtual media**

#. In the **SSH public key** text box privide the public key you created in this section :ref:`prov_vm`

   Make sure to copy the **public key**
   
   .. figure:: images/ocp_public_key.png

#. Click on **Generate Discovery ISO**

#. Copy the **Discovery ISO URL** and note it down somewhere. You will need this for your next section while creating infrastructure.

   .. figure:: images/ocp_iso_url.png

#. Click on **Close**

Provision OCP Infrastructure 
+++++++++++++++++++++++++++++

This section of the lab is done on using Linux Tools VM.

If it is not already present in your HPOC, create Linux Tools VM using instructions in :ref:`linux_tools_vm`

.. note::
 
 You are able to create these VMs and its resources using the GUI. But in this section we will use **Terraform** code for repeatability. 

#. Login to the Linux Tools VM using the following credentials
    
   - **Username** - root
   - **Password** - default password

#. Run the following commands in sequence

   .. code-block:: bash

    yum update -y 
    yum install -y yum-utils
    yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
    yum -y install terraform
    yum -y install git

#. Clone the following git repo and initialise Terraform provider

   .. code-block:: bash

    git clone https://github.com/nutanix-japan/tf-ocp-infra
    cd tf-ocp-infra
    alias "tf=terraform" 
    tf init

#. Get your variables file ready with your HPOC/SPOC environment information

   .. code-block:: bash
    
    cp terraform.tfvars.sample terraform.tfvars
     
#. Modify your variables to suit your HPOC environment using ``vi`` or ``nano`` editor

   .. code-block:: bash
    
    vi terraform.tfvars

   .. code-block:: bash 

    cluster_name        = "your cluster name" # << Change this
    subnet_name         = "your network name" # << Change this
    user                = "admin"             # << Change this
    password            = "XXXXXXX"           # << Change this
    endpoint            = "Prism Element IP"  # << Change this
    vm_worker_prefix    = "ocp-worker"
    vm_master_prefix    = "ocp-master"
    vm_domain           = "ntnxlab.local"
    vm_master_count     = 3
    vm_worker_count     = 2
    image_uri           = "Discover ISO URL you copied from RH Console" # << Change this
    
#. Validate and run your Terraform code

   .. code-block:: bash
    
    tf validate

    # if there are any validate errors fix or request support from your instructor
    
    tf apply 

    # Terraform will show you all resources that it will to create
    # Type yes to confirm 

   
   .. code-block:: bash

      # Sample output for the command above

      Terraform will perform the actions described above.
      Only 'yes' will be accepted to approve.

      Enter a value: yes

      nutanix_image.rhocs: Creating...
      nutanix_image.rhocs: Still creating... [10s elapsed]
      nutanix_image.rhocs: Creation complete after 14s [id=e04cff4e-a6cf-45f9-890d-96097c4b53ed]
      nutanix_virtual_machine.rhocs-master[0]: Creating...
      nutanix_virtual_machine.rhocs-master[1]: Creating...
      nutanix_virtual_machine.rhocs-worker[0]: Creating...
      nutanix_virtual_machine.rhocs-master[2]: Creating...
      nutanix_virtual_machine.rhocs-worker[1]: Creating...
      nutanix_virtual_machine.rhocs-master[0]: Still creating... [10s elapsed]
      nutanix_virtual_machine.rhocs-master[1]: Still creating... [10s elapsed]
      nutanix_virtual_machine.rhocs-master[2]: Still creating... [10s elapsed]
      nutanix_virtual_machine.rhocs-worker[0]: Still creating... [10s elapsed]
      nutanix_virtual_machine.rhocs-worker[1]: Still creating... [10s elapsed]
      nutanix_virtual_machine.rhocs-master[0]: Creation complete after 16s [id=3a88a3d7-304e-4284-886d-f7882764d7cc]
      nutanix_virtual_machine.rhocs-master[2]: Creation complete after 17s [id=5e87599a-5643-465d-9870-5b34751b2158]
      nutanix_virtual_machine.rhocs-worker[0]: Creation complete after 17s [id=78fb2e69-fee7-4244-ae5c-55ffbc1da21d]
      nutanix_virtual_machine.rhocs-master[1]: Creation complete after 17s [id=7775b527-fc55-4cac-aabc-a024ea4938c1]
      nutanix_virtual_machine.rhocs-worker[1]: Creation complete after 17s [id=c9801a82-a7e3-444e-a206-d5e3e3a75bb1]

      Apply complete! Resources: 6 added, 0 changed, 0 destroyed.

#. Run the Terraform state list command to verify what resrouces are created

   .. code-block:: bash
   
      tf state list

   .. code-block:: bash

      # Sample output for the command above

      data.nutanix_cluster.cluster            # < This is your existing Prism Element HPOC/SPOC cluster
      data.nutanix_subnet.subnet              # < This is your existing Primary subnet
      nutanix_image.rhocs                     # < This is OCP Discovery ISO image
      nutanix_virtual_machine.rhocs-master[0] # < This is master vm 1
      nutanix_virtual_machine.rhocs-master[1] # < This is master vm 2
      nutanix_virtual_machine.rhocs-master[2] # < This is master vm 3
      nutanix_virtual_machine.rhocs-worker[0] # < This is worker vm 1
      nutanix_virtual_machine.rhocs-worker[1] # < This is worker vm 2


#. Login to Prism Element :fa:`bars` > VM and verify the VMs and if they are powered on

   .. figure:: images/ocp_tf_vms.png

Install OCP Cluster in Red Hat Console
++++++++++++++++++++++++++++++++++++++

In this section we will use Red Hat Console's Assisted Installer wizard to install the OCP cluster with the VMs we have provisioned. 

#. Return to Red Hat Openshift Console and check if the VMs appear

   .. figure:: images/ocp_rh_console_vms.png

#. Assign IPs for your **API Virtual IP** and **Ingress Virtual IP** from you HPOC/SPOC **Primary** network pool

   .. figure:: images/ocp_ing_api_ips.png
   
   .. note::

    Prism Element now shows used IP in the Networking setup and you can use this to determine the IPs for **API Virtual IP** and **Ingress Virtual IP** of your OCP cluster.
    
    Prism Element > :fa:`bars` > Network Configuration > **Primary**

    .. figure:: images/pe_net_config.png

    A new pop-up will show the used IPs. Choose two **unused** IPs from here.

    .. figure:: images/pe_used_ips.png
   
#. In the **Host inventory** section, choose the **Control Plane Node** for Master VMs and **Worker** nodes for Worker VMs from the drop down menu

   .. figure:: images/ocp_node_roles.png

#. Click **Next** at the bottom of the page
   
#. Review your setup information and click on **Install Cluster**

   .. figure:: images/ocp_cluster_summary.png

#. You will be taken to monitoring your installation progress

   .. figure:: images/ocp_install_start.png

   Now the cluster deploy will proceed

#. Watch for any messages about user interactions in the progress page

   .. figure:: images/ocp_user_inter.png

#. This message is wanting the user to unmount the installation Discovery ISO so they VM can boot into the OS drive

#. Go to Prism Element > VM > Master/Worker VM > update

#. Under Disks > Click on Eject 

   .. figure:: images/pe_vm_cd_eject.png

#. Click on Save

#. Under **Power Off Actions** choose to Guest Reboot the VM where there are pending user action

#. Repeat ejecting CD-ROM for all VMs and rebooting it as the Wizard prompts for user action (do not do this before the prompting)

#. Once all the user actions are sustained for Master and Worker VMs, OCP cluster will be installed

#. You can access your installed OCP Cluster Manger page using the URL provided 

   .. figure:: images/ocp_install_finish.png

   .. note::

     This URL can only be accessed within your HPOC/SPOC environment

     You can access by creating DNS entry in your AD server or using IP address. 

     Click on **Not able to access Web Console?** link in the status page to reveal IP addresses and DNS entry suggestions.
     
     .. figure:: images/ocp_access.png

     .. figure:: images/ocp_dns_hosts.png

     You can download KUBECONFIG file to use ``oc`` commands.

#. After adding DNS/hosts entries to your environment, use Windows Tools PC you can browse to the IP addresses, or DNS entries you have manually created you are able to login to OC console.

   .. figure:: images/ocp_console_ai.png
     
You have successfully deployed a OCP cluster using Red Hat Console Assisted Installer. 







  






