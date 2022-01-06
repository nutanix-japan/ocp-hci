.. _ocp_csi:

-------------------------------
OCP: Deploy Nutanix CSI for OCP
-------------------------------

In this section we will enable your deployed OCP cluster to have access to Nutanix HCI storage in the form of StorageClass, Physical Volumes and Pysical Volume claims.

We will also install kubernetes operator provided by Nutanix to maintain the lifecycle of operator.

See `What are Kubernetes Operators? <https://docs.openshift.com/container-platform/4.7/operators/understanding/olm-what-operators-are.html>`_ for more information.

OpenShift provides a easy way of implementing these third-party operators via OperatorHub from the cluster manager GUI .

As the Nutanix CSI Operator provides the following capabilities as of now, we will install it to manage all our Nutanix StorageClass in this OCP cluster.

- Basic install
- Seamless upgrades (Manual or Automatic)

In future Nutanix CSI Operator may provide the following additional features:

- Full lifecycle
- Deep insights 
- Autopilot

Install Nutanix CSI Operator
++++++++++++++++++++++++++++

#. Login to you Windows Tools VM using the following credentials;

   - **Username** - administrator@ntnxlab.local
   - **Password** - nutanix/4u

#. Using Chrome browser browse to Console URL you obtained in the previous section. 

   .. code-block:: url
   
    https://console-openshift-console.apps.ocp1.ntnxlab.local

#. Use your credentials to Login
   
   - **Username** - kubeadmin
   - **Password** - password from previous section :ref:`ocp_cluster_login`

#. Click on **Log in** (if you are not already logged in)

#. Click on **Operator** > **Operator Hub**

#. You will see all third party operators categorised by solution driven use cases (e.g. Big Data, Database, etc)

   .. figure:: images/ocp_operator_hub.png

   This is in principle similar to Nutanix Calm Marketplace where you can request and install applications.

#. In the **Filter by Keyword** text box, type **Nutanix** to find the Nutanix CSI Operator

#. Click on the **Nutanix CSI Operator**, read basic information about the Operator and click on **Install**

   .. figure:: images/nutanix_csi_operator.png

   Also make sure to check the supported orchestration (RH OCP) platforms

#. In the Operator install wizard choose the following:

   - **Update channel** - stable 
   - **Operator recommended Namespace** - ntnx-system
   - **Update approval** - Automatic (admins usually choose Manual option where control over updates is necessary)

   .. figure:: images/ocp_operator_install_options.png
   
#. Click on **Install** 

#. Once installed you will see the operator in **Operator** > ** Installed Operators** 

You have succesfully installed the Nutanix CSI operator to take care of StorageClass installation and upgrades.

.. note::

 Operators can also be installed using ``oc`` commandline. For more information refer to OCP documentation `here. <https://docs.openshift.com/container-platform/4.7/operators/admin/olm-adding-operators-to-cluster.html#olm-installing-operator-from-operatorhub-using-cli_olm-adding-operators-to-a-cluster>`_

Install StorageClass
++++++++++++++++++++

#. In Calm go to your **Applications** > **Openshift ocp1** application

#. Go to the **Services** 

#. Select your **LB_DNS** service

#. Click on **Open Terminal**
   
   .. figure:: images/ocp_lbdns_terminal.png

   The terminal will open in a new browser tab

#. Export the OCP cluster's KUBECONFIG file to environment so we can perform ``oc`` commands

   .. code-block:: bash
      
    export KUBECONFIG=~/openshift/auth/kubeconfig

#. Create a secret that the StorageClass can use to access the Nutanix HCI storage

   Copy the following Secret configuration script, modify required fields and execute it in the command line

   **Open a text editor (notepad / vi / nano), paste the contents and change the fields indicated to suit your environment. Example is also provided**

   **Be sure to use your environment's details for the following fields:**

   - Prism Element IP
   - Prism Element UserName
   - Prism Element Password

   .. code-block:: bash

    cat << EOF | oc create -f -
    apiVersion: v1
    kind: Secret
    metadata:
      name: ntnx-secret
      namespace: ntnx-system
    stringData:
      key: <Prism Element IP>:9440:<Prism Element UserName>:<Prism Element Password>
      # example: 
      # key: 10.38.2.71:9440:admin:password
    EOF

#. Copy the following StorageClass configuration script, modify required fields and execute it in the command line
    
   **Open a text editor (notepad / vi / nano), paste the contents and change the fields indicated to suit your environment. Example is also provided**

   **Be sure to use your environment's details for the following fields:**

   - Data Services IP and 
   - Storage Container Name

   .. code-block:: bash

    cat << EOF | oc create -f -
    apiVersion: storage.k8s.io/v1
    kind: StorageClass
    metadata:
        name: nutanix-volume
    provisioner: csi.nutanix.com
    parameters:
      csi.storage.k8s.io/provisioner-secret-name: ntnx-secret
      csi.storage.k8s.io/provisioner-secret-namespace: ntnx-system
      csi.storage.k8s.io/node-publish-secret-name: ntnx-secret
      csi.storage.k8s.io/node-publish-secret-namespace: ntnx-system
      csi.storage.k8s.io/controller-expand-secret-name: ntnx-secret
      csi.storage.k8s.io/controller-expand-secret-namespace: ntnx-system
      csi.storage.k8s.io/fstype: ext4
      dataServiceEndPoint: <Data Services IP>:3260
      # example: 
      # dataServiceEndPoint: 10.38.5.72:3260
      storageContainer: <Storage Container Name>
      # example: 
      # storageContainer: Default
      storageType: NutanixVolumes
    allowVolumeExpansion: true
    reclaimPolicy: Delete
    EOF

#. List your StorageClass
 
   .. code-block:: bash

    oc get StorageClass -A
    # example output here
    # NAME             PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
    # nutanix-volume   csi.nutanix.com   Delete          Immediate           true                   45h

#. Create a subscription of the Operator to attach it to ``ntnx-system`` namespace

   .. code-block:: bash

    cat << EOF | oc create -f -
    apiVersion: crd.nutanix.com/v1alpha1
    kind: NutanixCsiStorage
    metadata:
        name: nutanixcsistorage
        namespace: ntnx-system
    spec:
        namespace: ntnx-system
    EOF
   
We have sucessfully installed Nutanix StorageClass so we can provision Physical Volumes (PV) and Physical Volume Claims (PVC) for the applications we will be deploying in this OCP cluster.



    





