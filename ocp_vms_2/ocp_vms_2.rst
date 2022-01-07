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

   - **Number of Workers** - 2
   - **OCP_PULL_SECRET** - download/copy pull secret this from your Red Hat Portal `Login <https://console.redhat.com/openshift/install/pull-secret>`_ (you will need Red Hat Portal Access)
   - **OCP_SUBDOMAIN** - Initials2 (e.g. **xyz2**) 
   - **OCP_MACHINE_NETWORK** - provide your Primary network address for your HPOC/SPOC (e.g. 10.38.2.64/26 ) - check in your cluster reservation email/webpage
   - **OCP_BP** - XYZ_OCP_Master_Worker
   
   .. note::

    Make sure that the **OCP_SUBDOMAIN** name is be unique for each OCP cluster you deploy with this blueprint. A DNS zone is created based on this subdomain name. 

    You can observe this in DNS in your AutoAD VM for other deployed clusters.

    .. figure:: images/ocp_xyz1_cluster_dns.png

   .. figure:: images/ocp_deploy_task_runtime_variables_2.png

#. Click on **Run**

#. Go to the **Audit** of your tab to check the stautus of the launched **Deploy OCP** action

   .. figure:: images/ocp_provvm_deploy_audit.png

#. If this action launch is succesful, you will see another application **Openshift xyz2** in the **Calm** > **Applications** in a **Provisioning** state
 
   .. figure:: images/ocp_app_provisioning.png

#. Click on the **Openshift xyz2** application and go to **Audit** tab

   .. figure:: images/ocp_deploy_audit_2.png

#. Observe the VM create, package and other Calm actions 

#. This should take about 20-25 minutes

#. Once deployed the Calm application **Openshift xyz2** will be in a running state

   .. figure:: images/ocp_prov_vm_audit.png

#. Go to DNS in your AutoAD VM to check for the **xyz2** subdomain for your **Openshift xyz2**
   
   .. note::
     
     If other users are using your cluster you will see other subdomains in DNS