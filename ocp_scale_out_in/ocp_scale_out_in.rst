.. _ocp_scale_out_in:

-------------------------------------
OCP: Day O and Day 1 Operations
-------------------------------------

In this section we will look at Typical Day 1 operations of your OCP cluster in terms of scaling out and scaling in VMs.

For the purposes of the lab, we will achieve scale in/out operations by triggering a Calm Action manually. These Calm Actions can also be triggered using REST calls from an automation script. 

.. _ocp_cluster_login:



Day 0: Getting OCP Cluster Login Details 
++++++++++++++++++++++++++++++++++++++++

#. In **Calm** > **Applications**

#. Click on the **Openshift ocp1** application and go to **Audit** tab

#. Go to the end of the audit trail and look for **OS_Status_Check Start** task and expand 

#. Click on **OS_Status_Check - Show Login Information** sub task and click to maximize the output window

   .. figure:: images/ocp_os_status_check.png

#. Make a note of the following:
 
   - Console URL
   - Credentials: Username / password 

   .. figure:: images/ocp_console_cred.png

   Note that the password will be different for your implementation. 

#. You will need these details to logon to the OCP cluster managment console

Day 0: Accessing OCP Cluster Manager
+++++++++++++++++++++++++++++++++++++

In the HPOC environment all the DNS lookups are contained within each HPOC cluster. 

For this reason, we will have to access the OCP Cluster Manager using the Windows Tools VM.

.. note:: 
 
  DNS lookup will not work from anywhere else except the Windows Tools VM

#. Login to you Windows Tools VM using the following credentials;

   - **Username** - administrator@ntnxlab.local
   - **Password** - nutanix/4u

#. Using Chrome browser browse to Console URL you obtained in the previous section. 

   .. code-block:: url
   
    https://console-openshift-console.apps.ocp1.ntnxlab.local

#. Use your credentials to Login
   
   - **Username** - kubeadmin
   - **Password** - password from previous section :ref:`ocp_cluster_login`

#. Click on **Log in**

   You will see the cluster manager's web interface. 

   This is in principle similar to Nutanix Karbon's individual kubernetes cluster management page. Note that you can only manage the resources of just this local cluster. 

   .. note::

    To be able to manage other clusters, you will need to register this cluster to ``https://console.redhat.com/openshift/`` web interface

   .. figure:: images/ocp_overview.png

#. Go through the following important components and make a note of the details:

   - Overview 
   - Projects
   - Networking
     - Services
     - Routes
   - Storage
     - StorageClasses
   - Monitoring
     - Dashboards
   - Compute
     - Nodes (these are your OCP Cluster VMs)
   - User Management (identify which users/user groups from your AutoAD are added)
     - Users
     - Groups
   - Administration
     - Cluster Settings - get your cluster's version
     - Namespaces - make a note of system and user namespaces 

You can see how managing all parts of the OCP cluster are in a single place. In a open source kubernetes cluster you would have to install many management packages and go many places them. This is one of the strong selling points of a OCP kubernetes cluster.

Day 0: Deploy a Simple POD
++++++++++++++++++++++++++

- Steps to deploy a simple pod 
- Expose it using services 


Day 1: Scale Out Worker Nodes
+++++++++++++++++++++++++++++

As any environment would require more kubernetes nodes from time to time to host more workloads. In this sectoin we will look at how a extra worker node is added to this OCP cluster.

We will do this using Calm Actions. 

#. In **Calm** > **Applications**

#. Click on the **Openshift ocp1** application and go to **Manage** tab

#. In the **Scale Out** action, click on :fa:`play` play button

   .. figure:: images/ocp_scaleout.png

#. Enter ``2`` as the **Scale out number of Workers** and click on **Run**

   .. figure:: images/ocp_scaleout_no.png

#. In the **Scale Out** action, click on the :fa:`eye` eye button to see the action's progression status
  
   .. figure:: images/ocp_scaleout_prog.png

#. Monitor the events in Prism Central to see what is being create on the HCI infrastructure

#. Note down the number of resources created (e.g VM, Disk, etc)

#. This will take about 5-10 minutes

#. Once the Scale Out action is successfully completed, go to the **Service** tab of the VM and check if two more nodes are added

   .. figure:: images/ocp_scaleout_complete.png

#. In your OCP Console, go to Compute > Nodes and make sure OCP has picked up the two extra worker nodes
   
   .. figure:: images/ocp_scaleout_check_cm.png

Calm makes it really simple to scale out your OCP cluster Day 1 operations. 

Day 1: Scale In Worker Nodes
++++++++++++++++++++++++++++

As any environment would require more kubernetes nodes from time to time to host more workloads. In this sectoin we will look at how a extra worker node is added to this OCP cluster.

We will do this using Calm Actions. 

#. In **Calm** > **Applications**

#. Click on the **Openshift ocp1** application and go to **Manage** tab

#. In the **Scale In** action, click on :fa:`play` play button

   .. figure:: images/ocp_scalein.png

#. Enter ``1`` as the **SCALEIN_WORKER** and click on **Run**

   .. figure:: images/ocp_scalein_no.png

#. In the **Scale In** action, click on the :fa:`eye` eye button to see the action's progression status
  
   .. figure:: images/ocp_scalein_prog.png

#. Monitor the events in Prism Central to see what is being create on the HCI infrastructure

#. Note down the number of resources created (e.g VM, Disk, etc)

#. This will take about 5-10 minutes

#. Once the Scale Out action is successfully completed, go to the **Service** tab of the VM and check if one worker nodes is deleted

   .. figure:: images/ocp_scalein_complete.png

#. In your OCP Console, go to Compute > Nodes and make sure OCP has picked up the two extra worker nodes
   
   .. figure:: images/ocp_scalein_check_cm.png

   .. note::

    OCP keeps the node entry as **Not Ready** (unless manually deleted) to register a new node that comes in as a result of another scale in 

    **Tip** - Run the Scale Out action once again with ``1`` node and check in the OCP node becomes ready.

Calm makes it really simple to scale in your OCP cluster Day 1 operations as the resource requirements reduces. 

Takeaways
+++++++++

- Calm makes Day 0 and Day 1 operations easy to manage in a complex environment
- As maintaining a OCP environment is difficult in terms of administration, Calm helps facilitate this in a repeatable and reliable manner
