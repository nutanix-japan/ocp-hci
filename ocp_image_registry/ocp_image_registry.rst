

.. _ocp_image_registry:

----------------------------
OCP: Create Image Registry
----------------------------

OCP comes with a in-built container image registry. The advantages of these container image registry are the following:

- Integrates with OCP clusters authentication and authorization system 
- Provides a image source local to the cluster for all running workloads
- Integrates with OCP clusters CICD workflows - for example when a new cotainer images is uploaded to this registry, the cluster can update the image of all the running running contianers
- Registry can be easily scaled up or down
- Image data is stored in a cloud storage 

In this lab we will create a physical volume claim (PVC) and present it as storage to the running image registry on this OCP cluster. This will be our first use case for Nutanix provided HCI storage.

The PVC will be 100 GB.

Creating the PVC
++++++++++++++++

#. In Calm go to your **Applications** > **Openshift ocp1** application

#. Go to the **Services** 

#. Select your **LB_DNS** service

#. Click on **Open Terminal**
   
   .. figure:: images/ocp_lbdns_terminal.png

   The terminal will open in a new browser tab

#. Copy the following StorageClass configuration script and paste it in the command line window
    
   .. code-block:: bash
    
    cat << EOF | oc apply -f -
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: image-registry-claim
      namespace: openshift-image-registry
    spec:
      accessModes:
      - ReadWriteOnce
      resources:
        requests:
          storage: 100Gi
      storageClassName: nutanix-volume
    EOF

   .. code-block:: bash

    # example output here for the above command
    # persistentvolumeclaim/image-registry-claim created

Presenting the PVC to OCP Image Registry
+++++++++++++++++++++++++++++++++++++

#. Copy the following patch configuration script and paste it in the command line window
  
   .. code-block:: bash
  
    oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed","storage":{"pvc":{"claim":"image-registry-claim"}},"rolloutStrategy": "Recreate"}}'
   
   .. code-block:: bash

    # example output here for the above command
    # config.imageregistry.operator.openshift.io/cluster patched

   This will patch the image registry with the created storage (PVC) by re-creating imageregistry operator.

#. To see the changes at the operator level execute the following command:

   .. code-block:: bash
  
    oc describe configs.imageregistry.operator.openshift.io

   .. code-block:: bash

     # example output here for the above command
     Spec:
        Operator Log Level:  Normal
        Proxy:
        Replicas:  1
        Requests:
            Read:
            Max Wait In Queue:  0s
            Write:
            Max Wait In Queue:  0s
        Rollout Strategy:       Recreate
        Storage:
            Management State:  Unmanaged
            Pvc:
              Claim: image-registry-claim   ## << Here is the claim
        Unsupported Config Overrides:  <nil>

To check the existence this PV in Prism Element, check the volume detail in Storage container.

#. Go to your **Prism Element** > **Storage** > **Volume Group**

#. You should see a 100 GB disk provisioned as shown below 
   
   .. figure:: images/ocp_pv_vg.png

You have successfully created a storage PVC in Nutanix HCI and presented it to a resource in OCP cluster. We will create other resources and present Nutanix HCI storage to them in the subsequent sections of the lab.

    

