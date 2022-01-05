.. _import_bp:

-------------------------------
OCP: Import OCP Calm Blueprints
-------------------------------

In this lab, we will set up OCP infrastructure using Nutanix Calm. 

Calm provides an easy way of provisioning and configuring repeatable infrastructure. Once the blueprint is developed, it can be used repeatedly and constantly improved.

As OCP infrastructure requirements consist of many moving parts, it is easy to manage the soltuion in Calm. Since Calm can be used to deploy the same infrastructure accross multiple clouds, it is a optimal solution to manage the lifecycle of OCP infrastructure.

At a high-level OCP consists of the following components.

1. Boostrap VM
2. Stacked Master Nodes (Kubernetes Master + ETCD)
3. Worker Nodes (used to run user workloads)
4. Infra Nodes (used to run infrastructure based workloads)

.. figure:: images/ocp_infra.png 

We will use two Calm blueprints to deploy OCP infrastructure.

.. note::

 These Calm blueprints are developed by Nutanix's very own SA **Wolfgang Huse** and team. For updates and more information about these blueprints check this github page `here. <https://github.com/nutanix/openshift/tree/calm-automation/automation/calm>`_

1. Blueprint 1 - provisions a OCP provisioning VM with Calm action to deploy OCP 
2. Blueprint 2 - is called by a Calm action in ``Blueprint 1`` with all the necessary information that ``Blueprint 2`` needs to run

.. figure:: images/ocp_bp_actions.png 

Pre-requisites for OCP Environment
++++++++++++++++++++++++++++++++++

OCP has the following infrastructure pre-requistes available in the environment. Since we will be doing the labs in Nutanix HPOC/SPOC clusters, here is a list of OCP pre-requisites and HPOC/SPOC components that fulfill them.

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
    - CentOS, RHOCS, etc.
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

Upload Blueprints
+++++++++++++++++

#. Download `Provisioning VM Blueprint 1 <https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/import_bp/OCP-Master-Worker.json>`_ (right-click and save)

#. Download `OCP Master Worker Blueprint 2 <https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/import_bp/OCP-Prov-VM.json>`_ (right-click and save)

#. In **Prism Central** > **Services** > **Calm**
 
#. Click on **Blueprints**

#. Select your Provisioning VM ``Blueprint 1``

#. Fill the following Details 

   - **Blueprint Name** - OCP_Prov_VM
   - **Project** - BootcampInfra

   .. figure:: images/ocp_upload_bp.png

#. Click on **Upload**

#. Return to **Blueprints** and upload OC Master Worker ``Blueprint 2``
   
   - **Blueprint Name** - OCP_Master_Worker
   - **Project** - BootcampInfra

#. Return to **Blueprints** once again and you will see both blueprints uploaded to Calm

   .. figure:: images/ocp_bp_list.png

