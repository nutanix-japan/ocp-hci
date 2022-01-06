.. _ocp_vms_2:

-----------------------------------
OCP: Create Additional OCP Clusters
-----------------------------------

There will instances where you will want to deploy a second OCP cluster in the same/different environment using these blueprints.

We are able to achieve this by executing the **Deploy OCP** action once again from the Provisioning VM


#. In **Prism Central** > **Services** > **Calm**
 
#. Click on **Applications**

#. Select your Provisioning VM ``Initials_PROV_VM`` application
 
   .. figure:: images/prov_vm_app.png
   
#. Click on **Manage** tab 

#. Click on the play :fa:`play` button next to **Deploy OCP** action

   .. figure:: images/ocp_deployocp_action.png

#. Fill only the following details:

   - **HPOC_USER_INITIALS** - XYZ
   - **Number of Workers** - 2
   - **OCP_PULL_SECRET** - download/copy this from your Red Hat Portal `Login <https://console.redhat.com/openshift/install/pull-secret>`_ (you will need Red Hat Portal Access)
   - **OCP Subdomain** - ocp2 (this must be unique from your previous OCP cluster)
   - **OCP_MACHINE_NETWORK** - provide your Primary network address for your HPOC/SPOC (e.g. 10.38.2.64/26 ) - check in your cluster reservation email/webpage
   - **OCP_BP** - OCP_Master_Worker (modify this if you have used a different blueprint name)

#. Click on **Run**

   .. figure:: images/ocp_deploy_task_runtime_variables_2.png

#. Go to the **Audit** tab to check the stautus of the launched **Deploy OCP** action

   .. figure:: images/ocp_deploy_audit.png

#. If this action launch is succesful, you will see another application **Openshift ocp2** in the **Calm** > **Applications** in a **Provisioning** state
 
   .. figure:: images/ocp_app_provisioning.png

#. Click on the **Openshift ocp2** application and go to **Audit** tab

#. Observe the vm create, package and other Calm actions 

#. This should take about 20-25 minutes

#. Once deployed the Calm application **Openshift ocp2** will be in a running state

   .. figure:: images/ocp_prov_vm_audit.png