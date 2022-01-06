.. _ocp_vms:

-------------------------------
OCP: Deploy OCP Infrastructure
-------------------------------

In this section we will deploy a OCP cluster using Nutanix Calm.

As discussed in the previous section about the Calm blueprints, 

1. Blueprint 1 - provisions a OCP provisioning VM with Calm action to deploy OCP
2. Blueprint 2 - is called by a Calm action in ``Blueprint 1`` with all the necessary information that ``Blueprint 2`` needs to run

Since we sucessfully setup OCP provisioning VM, we have to do the following to proceed:

1. Configure OCP VM Blueprint for environment information
2. Call the ``Deploy OCP`` action in Blueprint 1 to deploy OCP cluster


Configuring OCP VM Blueprint
++++++++++++++++++++++++++++

In this section we will configure the environemnt for the OCP VM blueprint to run. This will include the following:

- Credentials (SSH Private Key) for the services
- Confirming OCP DNS endpoint for the services 
- NICs are auto selected during execution 

.. note::

  Latest versions of Calm automatically assigns endpoint to Calm Actions if only one is present. It is important to confirm this in the lab.

#. In **Prism Central** > **Services** > **Calm**
 
#. Click on **Blueprints**
 
We need to configure network and credentials for the blueprint so it can be deployed in your HPOC/SPOC.

#. Select your Provisioning VM ``OCP_Master_Worker`` blueprint

#. Click on **Credentials** (next to Save)

#. Copy and paste the following primary key (this could be any primary key that you may already have)
   
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

#. Click on **Save**

   .. figure:: images/ocp_bp_save_cred.png

#. Click on **Back** to return to the blueprints main window

#. You will notice a few warnings for KUBEADMIN and KUBECONFIG variables. These can be ignored as they will be auto-generated.

Now we will check if OCP DNS endpoints are configured for the services.

#. In the blueprint's main window, expand the application and services pane

#. Click on the **Worker** service and select the **Create DNS Entry** action

   .. figure:: images/ocp_service_ep_confirm.png
   
#. Confirm that the enpoint is the **OCP DNS Integration** that you previously configured in :ref:`endpoint` section

#. Confirm  the **OCP DNS Integration**  for **Delete DNS Entry** task

#. Confirm the **OCP DNS Integration** endpoints for **Create DNS Entry** and **Delete DNS Entry** for the following services:

   - Bootstrap
   - Master

#. If the endpoint is not assigned properly, correct the condition

#. Save the blueprint and confirm there are no validation errors (warnings about KUBECONFIG and PE_PW (runtime variables) can be ignored)

   .. figure:: images/ocp_bp_save_validation.png

Deploying OCP Cluster 
+++++++++++++++++++++

#. In **Prism Central** > **Services** > **Calm**
 
#. Click on **Applications**

#. Select your Provisioning VM ``*Initials*_PROV_VM`` application
 
   .. figure:: images/prov_vm_app.png
   
#. Click on **Manage** tab 

#. Click on the play :fa:`play` button next to **Deploy OCP** action

   .. figure:: images/ocp_deployocp_action.png

#. This will open up **Run Action: Deploy OCP** window for user inputs

#. Fill only the following details:
   
   - **HPOC_USER_INITIALS** - XYZ
   - **Number of Workers** - 2
   - **OCP_PULL_SECRET** - download/copy pull secret this from your Red Hat Portal `Login <https://console.redhat.com/openshift/install/pull-secret>`_ (you will need Red Hat Portal Access)
   - **OCP_MACHINE_NETWORK** - provide your Primary network address for your HPOC/SPOC (e.g. 10.38.2.64/26 ) - check in your cluster reservation email/webpage
   - **OCP_BP** - OCP_Master_Worker (modify this if you have used a different blueprint name)

#. Click on **Run**

   .. figure:: images/ocp_deploy_task_runtime_variables.png

#. Go to the **Audit** tab to check the stautus of the launched **Deploy OCP**　action

   .. figure:: images/ocp_deploy_audit.png

#. If this action launch is succesful, you will see another application **Openshift ocp1** in the **Calm** > **Applications** in a **Provisioning** state
 
   .. figure:: images/ocp_app_provisioning.png

#. Click on the **Openshift ocp1** application and go to **Audit** tab

#. Observe the vm create, package and other Calm actions 

#. This should take about 20-25 minutes

#. Meanwhile you can notice that Prism Central's event will be getting quite busy. This is a result of VMs, storage, etc,. getting provisioned by Calm using REST calls to Prism Cental

#. Once deployed the Calm application **Openshift ocp1** will be in a running state

   .. figure:: images/ocp_prov_vm_audit.png

Now that your OCP cluster has been deployed, in the **Services** tab, lets check the VMs (Services in Calm) and their corresponding functions.

Click on each VM (Service) will display IP addresses and Open Terminal options. We will be using the private key to ssh into these VMs (if required)

.. figure:: images/ocp_deployed_vms.png

We have completed this section of the lab where our OCP cluster is sucessfully deployed.

.. figure:: images/ocp_vm_complete.png

We will proceed to test Day 1 operations (Scale out and Scale in) in the next section of the lab.
