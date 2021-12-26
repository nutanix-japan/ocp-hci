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

1. Blueprint 1 - provisions a management VM with Calm action to deploy OCP 
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
    - AutoAD (Windows based Active Directory and DNS server)
    - Deployed with HPOC 
  * - Calm Project
    - BootcampInfra
    - Deployed with HPOC
  * - VM Boot images
    - CentOS, RHOCS, etc.
    - Deployed with HPOC (downloadable images)
  * - DNS Entries Update Mechanism
    - Calm Endpoint to AutoAD and DNS server
    - You will create this


Now we can proceed to setup pre-requistes and deploying blueprints. 

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

#. Download `OCP Master Worker Blueprint 2 <https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/import_bp/OCP-Master-Worker.json>`_ (right-click and save)

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

Deploying Provisioning VM 
++++++++++++++++++++++++++

Configuring Provisioning VM Blueprint
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now lets configure and deploy Provisioning VM

#. In **Prism Central** > **Services** > **Calm**
 
#. Click on **Blueprints**

#. Select your Provisioning VM ``OCP_Prov_VM`` blueprint

We need to configure network and credentials for the blueprint so it can be deployed in your HPOC/SPOC.

#. Select the Provisioning_VM service 

#. In the services' VM properties, Choose Primary as the NIC 1

   .. figure:: images/ocp_prov_vm_nic1.png

#. Click on **Save**
   
   .. figure:: images/ocp_bp_save.png

#. Copy the following primary key (this could be any primary key that you may already have)
   
   .. code-block:: bash

    -----BEGIN OPENSSH PRIVATE KEY-----
    b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
    NhAAAAAwEAAQAAAQEA5//EaGHrEDLtng7U252IFsWvglORQXCGCpY3urfMOjl/LMY+41s7
    ug028XPlMO+kw5J6ylIgcepWijrfyqBTklHjXVNWo0WuukjrbEwG2CNHwdaCAGwBXhzNHV
    kgiaMj9mCCoNhZ6UZW+X2WsdpSjgkTKKXGoLh6R5wm+5ZdXvsjlZcwrNvuwKJ8uAsdWafV
    KlafThHUNHpLh+ZvNiinLGmfiBx1DTZzIUEsjf4fYcRk1drUbJnrMO6eXHPa6aPjx/MO9k
    eF6VLOM5dqZrdAYedMU3MFxWuTyXB6o6kd4ddmd/twSTC7cgRcADtaE5zcB3ZNmz2Ek5Lh
    dkIm9Yu8HwAAA9AATH8HAEx/BwAAAAdzc2gtcnNhAAABAQDn/8RoYesQMu2eDtTbnYgWxa
    +CU5FBcIYKlje6t8w6OX8sxj7jWzu6DTbxc+Uw76TDknrKUiBx6laKOt/KoFOSUeNdU1aj
    Ra66SOtsTAbYI0fB1oIAbAFeHM0dWSCJoyP2YIKg2FnpRlb5fZax2lKOCRMopcaguHpHnC
    b7ll1e+yOVlzCs2+7Aony4Cx1Zp9UqVp9OEdQ0ekuH5m82KKcsaZ+IHHUNNnMhQSyN/h9h
    xGTV2tRsmesw7p5cc9rpo+PH8w72R4XpUs4zl2pmt0Bh50xTcwXFa5PJcHqjqR3h12Z3+3
    BJMLtyBFwAO1oTnNwHdk2bPYSTkuF2Qib1i7wfAAAAAwEAAQAAAQEA59REnw4iB+jsW4J0
    oBSWFjNrA6+gOiVIPLIJdHMQCKrF86cS1rynPW7RFYx7XWBLZAcu3pgcxdimq7DTSrbVbC
    SaL/sEW1lZldsiLfD28eqnXQKOm/9Rior7SLV0dzVnzV55imNHzXy1ktA10vnYfwiAdaDa
    kGZoAsD+j2+dVDd3+qCLftw/uvXrlp8gx+bvX8xcoMxsckdKIQUE8zZJedV3QBfCHSnagc
    ub3BI3x5zaYTx3lZqaYYknkDr9gyGHuvPywJTMUsoAXhvt3/AONByLOUwpNt92svR+ljwy
    wszP7XqLtZEFA/Xb0t8fHd2ZhtndR9V5wTz6fPu/xvO6MQAAAIBQU03ZxyZyt72FygB/n+
    hBnVru7B7e3m2EjaLW6HKWmUaHez3go2GjAfQFMXHtHKkUbZ/25JwFNIYPRqgan45sgrxl
    UYGPCcwYMMqAg+8YsL8sxl6Ypc5YyoSUZTjRScGZw8us5qV/TBaXiY0TK3dqNM0VI7LFj6
    t89iKi6/KeZQAAAIEA+y0rIun5o27XfB+S4biP8xV1P8dls8NFg1uwFs50dJQfSozokW8z
    lgY92LvjHzgVs8OCj2HfrcM3+ySj6sZgRHkr+eL0fdum5ttYcuJx6391nuE4FehS5jtyjG
    3FIjVW9B5PPx7MXucp9w9ewSVNFo7Fn41d6++ALFThF1c2jucAAACBAOx0UbbNg4BTUKGU
    e8jg/uY36BFUJ8znKrPqLxMZv6YCW90dozPY0bFoV2iaPaqtKDJi/zrfyiteQkPA5o2MCt
    EEa8CaSUJaybyqTLwPfPX02U8jEwUF8n/5NGusMer8cIy0OFe1WyVEuaUk6zWgF7HIn/ba
    impp30n2vf7rkNoJAAAAFGxha3NobWlAQzAyRjcwRkxNTDg1AQIDBAUG
    -----END OPENSSH PRIVATE KEY-----

#. Click on **Credentials** (next to Save)

#. Paste the copied ssh private key 

#. Click on **Save**

   .. figure:: images/ocp_bp_save_cred.png

#. Click on **Back** to return to the blueprints main window

#. You will notice a few warnings for KUBEADMIN and KUBECONFIG variables. These can be ignored as they will be auto-generated.

   .. figure:: images/ocp_bp_warnings.png


Launching Provisioning VM Blueprint
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now it is time to launch the provisioning VM blueprint.

#. Click on **Launch** button

#. Give the application a name *Initials*_Prov_VM

   .. figure:: images/ocp_prov_vm_bp_launch.png

#. Click on **Deploy**

#. Go to Audit and check the deployment tasks

#. This should take about 10 minutes









