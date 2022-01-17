.. _ocp_assisted_install:

-----------------------------------------
OCP: RH Console Assisted Installation 
-----------------------------------------

In this optional lab we will go through Assited Installation feature that Red Hat Console offers.

Assisted Installer is a GUI driven way of deploying OCP clusters:

Pre-requisites for Assisted Installation
+++++++++++++++++++++++++++++++++++++++++

- Administrator creates the OCP Master and Worker VMs
- Adminsitrator provisions compute, networking and storage associated with the OCP Master and Worker VMs
- Access to Red Hat Console to continue with this lab.

Assisted Installer does the following:

- Provides RHOCS and OCP installation binaries in a CD-ROM ISO file
- Once the OCP VMs (Master and Worker) nodes are booted with this CD-ROM ISO file the installation begins
- Red Hat Console will manage and monitor the installation process from start to end 
- Upon successfull installatino of a OCP cluster the following will be provided:
  
  - KUBECONFIG file for ``oc`` access
  - DNS entries for OCP Cluster Manager access 

.. note::

  Assisted Installers usage is in `Technical Preview <https://access.redhat.com/support/offerings/techpreview>`_ stage with Red Hat. 

At a very high level, we will accomplish the following to get a OCP cluster deployed using Assisted Installer:

1. Provision OCP Cluster in Red Hat Console and Generate CD-ROM ISO URL 
3. Provision OCP Infrastructure - Create Master and Worker VMs in your AHV HPOC/SPOC cluster using Terraform infrastructure as code
4. Install OCP Cluster in Red Hat Console

Provision OCP Cluster in Red Hat Console
+++++++++++++++++++++++++++++++++++++++++

#. Go to ` Red Hat Console <https://console.redhat.com/openshift/assisted-installer/clusters>`_

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

This section of the lab is done on using Linux Tools VM 

If it is not already present in your HPOC, create Linux Tools VM using instructions in :ref:`linux_tools_vm`

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
    terraform init

#. Get your variables Ready

   .. code-block:: bash
    
    cp variables.tfvars.sample terraform.tfvars
     
#. Modify your variables to suit your HPOC environment using ``vi`` or ``nano`` editor

   .. code-block:: bash
    
    vi terraform.tfvars

   .. code-block:: bash 

    cluster_name        = "your cluster name" # << Change this
    subnet_name         = "your network name" # << Change this
    user                = "admin"             # << Change this
    password            = ""                  # << Change this
    endpoint            = "Prism Element IP"  # << Change this
    vm_worker_prefix    = "ocp-worker"
    vm_master_prefix    = "ocp-master"
    vm_domain           = "ntnxlab.local"
    vm_master_count     = 3
    vm_worker_count     = 2
    image_uri           = "uri you copied from RH Console while creating Assisted Installer Cluster" # << Change this
    
#. Run your Terraform code

   .. code-block:: bash
    
    tf validate
    tf apply 

    # Terraform will show you all resources that it will to create
    # Type yes to confirm 

    
   


    


    







