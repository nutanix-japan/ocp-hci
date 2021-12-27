.. _ocp_vms:

-------------------------------
OCP: Deploy OCP Infrastructure
-------------------------------

In this section we will deploy a OCP cluster using Nutanix Calm.

As discussed in the previous section about the Calm blueprints, 

1. Blueprint 1 - provisions a OCP provisioning VM with Calm action to deploy OCP
2. Blueprint 2 - is called by a Calm action in ``Blueprint 1`` with all the necessary information that ``Blueprint 2`` needs to run

Since we sucessfully setup OCP provisioning VM, we can proceed to call the ``Deploy OCP`` action in Blueprint 1

Now lets configure and deploy Provisioning VM

#. In **Prism Central** > **Services** > **Calm**
 
#. Click on **Applications**

#. Select your Provisioning VM ``*Initials*_PROV_VM`` application
 
   .. figure:: images/prov_vm_app.png
   
#. Click on **Manage** tab 

#. Click on the play :fa:`play` button next to **Deploy OCP** action

   prov_vm_deploy_ocp_action.png


