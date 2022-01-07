.. _ocp_vms:

-------------------------------
OCP: Deploy OCP Infrastructure
-------------------------------

In this section we will deploy a OCP cluster using Nutanix Calm.

As discussed in the previous section about the Calm blueprints, 

1. Blueprint 1 (XYZ_OCP_Prov_VM) - provisions a OCP provisioning VM with Calm action to deploy OCP
2. Blueprint 2 (XYZ_OCP_Master_Worker) - is called by a Calm action in ``Blueprint 1`` with all the necessary information that ``Blueprint 2`` needs to run

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

#. Select your Provisioning VM ``XYZ_OCP_Master_Worker`` blueprint

#. Click on **Credentials** (next to Save)

#. Copy the same **Private Key** that you generated from the previous section :ref:`prov_vm` and paste the generated primary key in the bluprints credential called **CRED**

   .. figure:: images/ocp_bp_save_cred.png

#. Click on **Save**

#. Click on **Back** to return to the blueprints main window

#. You will notice a few warnings for KUBEADMIN and KUBECONFIG variables. These can be ignored as they will be auto-generated.

Now we will check if OCP DNS endpoints are configured for the services.

#. In the blueprint's main window, expand the application and services pane

#. Click on the **Worker** service and select the **Create DNS Entry** action

   .. figure:: images/ocp_service_ep_confirm.png
   
#. Confirm that the endpoint is the **OCP DNS Integration** that you previously configured in :ref:`endpoint` section

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
   
   - **Number of Workers** - 2
   - **OCP_PULL_SECRET** - download/copy pull secret this from your Red Hat Portal `Login <https://console.redhat.com/openshift/install/pull-secret>`_ (you will need Red Hat Portal Access)
   - **OCP_SUBDOMAIN** - Initials1 (e.g. **xyz1**) 
   - **OCP_MACHINE_NETWORK** - provide your Primary network address for your HPOC/SPOC (e.g. 10.38.2.64/26 ) - check in your cluster reservation email/webpage
   - **OCP_BP** - XYZ_OCP_Master_Worker

   .. note::

    Make sure that the **OCP_SUBDOMAIN** name is be unique for each OCP cluster you deploy with this blueprint. A DNS zone is created based on this subdomain name. 

    You can observe this in DNS in your AutoAD VM once a OCP cluster deployed.

    .. figure:: images/ocp_xyz1_cluster_dns.png

   .. figure:: images/ocp_deploy_task_runtime_variables.png

#. Click on **Run**

#. Go to the **Audit** tab to check the stautus of the launched **Deploy OCP** action

   .. figure:: images/ocp_deploy_audit.png

#. If this action launch is succesful, you will see another application **Openshift xyz1** in the **Calm** > **Applications** in a **Provisioning** state (this might take up to 5 minutes to show up in the Web UI)
 
   .. figure:: images/ocp_app_provisioning.png

#. Click on the **Openshift xyz1** application and go to **Audit** tab

#. Observe the VM create, package install and other Calm actions 

#. This should take about 20-25 minutes

#. Meanwhile you can notice that Prism Central's event will be getting quite busy. This is a result of VMs, storage, etc,. getting provisioned by Calm using REST calls to Prism Cental

#. Once deployed the Calm application **Openshift ocp1 XYZ** will be in a running state

   .. figure:: images/ocp_prov_vm_audit.png

Now that your OCP cluster has been deployed, in the **Services** tab, lets explore the VMs (Services in Calm) and their corresponding functions.

Click on each VM (Service) will display IP addresses and **Open Terminal** options. We will be using the private key to ssh into these VMs (if required)

.. figure:: images/ocp_deployed_vms.png

We have completed this section of the lab where our OCP cluster is sucessfully deployed.

.. figure:: images/ocp_vm_complete.png

We will proceed to test Day 1 operations (Scale out and Scale in) in the next section of the lab.
