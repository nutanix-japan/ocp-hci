.. _import_bp:

-----------------------------------
OCP: Architecture and Initial Setup
-----------------------------------

In this lab, we will set up OCP infrastructure using Nutanix Calm. 

Calm provides an easy way of provisioning and configuring repeatable infrastructure. Once the blueprint is developed, it can be used repeatedly and constantly improved.

As OCP infrastructure requirements consist of many moving parts, it is easy to manage the soltuion in Calm. Since Calm can be used to deploy the same infrastructure accross multiple clouds, it is a optimal solution to manage the lifecycle of OCP infrastructure.

At a high-level OCP consists of the following components:

1. Boostrap VM
2. Stacked Master Nodes (Kubernetes Master + ETCD)
3. Worker Nodes (used to run user workloads)
4. Infra Nodes (used to run infrastructure based workloads)

.. figure:: images/ocp_infra.png 

Minimum Hardware Requirements for OCP VMs 
+++++++++++++++++++++++++++++++++++++++++

These are minimum required resrouces and software for the OCP VMs at the time of writing this bootcamp. 

The Calm blueprints in this lab will have these resources pre-configured for your use.

For latest resource requirements refer to OpenShift portal `here. <https://docs.openshift.com/container-platform/4.9/installing/installing_platform_agnostic/installing-platform-agnostic.html#installation-minimum-resource-requirements_installing-platform-agnostic>`_

.. list-table::
     :widths: 15 25 10 10 10 10
     :header-rows: 1

     * - OCP Role
       - Operating System
       - vCPU
       - RAM (minimum)
       - Storage
       - IOPS
     * - Bootstrap
       - RHCOS
       - 4
       - 16 GB
       - 100 GB
       - 300
     * - Master
       - RHCOS
       - 4
       - 16 GB
       - 100 GB
       - 300
     * - Worker
       - RHCOS, RHEL 7.9, or RHEL 8.4
       - 8
       - 16 GB
       - 100 GB
       - 300
       

We will use two Calm blueprints to deploy OCP infrastructure.

.. note::

 These Calm blueprints are developed by Nutanix's very own SA **Wolfgang Huse** and team. For updates and more information about these blueprints check this github page `here. <https://github.com/nutanix/openshift/tree/calm-automation/automation/calm>`_
 
 This lab uses a modified version of these blueprints to suit HPOC/SPOC environment and to achieve the learning objectives. 

1. Blueprint 1 - provisions a OCP provisioning VM with Calm action to deploy OCP 
2. Blueprint 2 - is called by a Calm action in ``Blueprint 1`` with all the necessary information that ``Blueprint 2`` needs to run

.. figure:: images/ocp_bp_actions.png 

Pre-requisites for Infrastructure in HPOC/SPOC Clusters
+++++++++++++++++++++++++++++++++++++++++++++++++++++++

Since we will be doing the OCP labs in Nutanix HPOC/SPOC clusters, here is a list of pre-requisites and HPOC/SPOC components that fulfill them.

Some are created with your HPOC/SPOC cluster runbook (see screenshot below) you chose while booking the cluster in `RX <https://rx.corp.nutanix.com/>`_

.. figure:: images/ocp_runbook.png

.. list-table::
  :widths: 40 70 40 
  :header-rows: 1

  * - OCP Pre-requisite
    - HPOC Component 
    - Status 
  * - IPAM Enabled Subnet
    - Primary Network
    - Deployed with HPOC
  * - Existing AD and DNS Server
    - AutoAD (Windows based Active Directory+DNS server)
    - Deployed with HPOC 
  * - Calm Project
    - BootcampInfra
    - Deployed with HPOC
  * - VM Boot images
    - CentOS, RHCOS, etc.
    - Deployed with HPOC (downloadable images)
  * - DNS Entries Update Mechanism
    - Calm Endpoint to contact AutoAD+DNS server
    - You will create this

Now we can proceed to setup pre-requistes and deploying blueprints. 

.. _endpoint:

Set up Calm Endpoint
+++++++++++++++++++++

Let's create a Calm endpoint for authentication, authorization and DNS updates for OCP cluster. 

#. In **Prism Central** > **Services** > **Calm**

#. Click on **Endpoint**

#. Click on **+Create Endpoint**

   .. figure:: images/ocp_calm_endpoint.png

#. Fill in the following fields:

   - **Name** - OCP DNS Integration
   - **Description** - (optional)
   - **Project** - BootcampInfra
   - **Type** -  Windows
   - **Target Type** - IP Addresses
   - **IP Address** - Your AutoAD VM IP (get from list of VMs)
   - **Connection Protocol** -  HTTP (default)
   - **Port** - 5985 (default)
   - **Username** - administrator@ntnxlab.local
   - **Secret Type** - Password
   - **Password** - nutanix/4u
   
   .. figure:: images/ocp_create_endpoint.png

#. Click **Save**


.. note::
 
  .. raw:: html

   <body><font color="green">Validation Task 1 - Take a screenshot of the Calm endpoint you just configured and store it somewhere safe. You will zip up all screenshots and upload to XReady mission after completing the lab. </font></body>

Upload Blueprints
+++++++++++++++++

#. Download blueprint1 by right-clicking on this link `Provisioning VM <https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/import_bp/XYZ_OCP_Prov_VM.json>`_ and save as Initials_OCP_Prov_VM.json file (e.g XYZ_OCP_Prov_VM.json)

#. Download blueprint2 by right-clicking on this link `OCP Master Worker <https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/import_bp/XYZ_OCP_Master_Worker.json>`_ and save as Initials_OCP_Master_Worker.json file (e.g XYZ_OCP_Master_Worker.json)

#. In **Prism Central** > **Services** > **Calm**
 
#. Click on **Blueprints**

#. Select your Provisioning VM Initials_OCP_Prov_VM.json blueprint

#. Fill the following Details 

   - **Blueprint Name** - XYZ_OCP_Prov_VM
   - **Project** - BootcampInfra

   .. figure:: images/ocp_upload_bp.png

#. Click on **Upload**

#. Return to **Blueprints** and upload OC Master Worker Initials_OCP_Master_Worker.json blueprint
   
   - **Blueprint Name** - XYZ_OCP_Master_Worker
   - **Project** - BootcampInfra

   .. figure:: images/ocp_upload_bp_mw.png

#. Return to **Blueprints** once again and you will see both blueprints uploaded to Calm

   .. figure:: images/ocp_bp_list.png

.. note::
 
  .. raw:: html

   <body><font color="green">Validation Task 2 - Take a screenshot of the uploaded blueprints. </font></body>


Now that we are setup with all required information. We can proceed to deploying the Provisioning VM and ultimately OCP Cluster in the next sections of the lab.


