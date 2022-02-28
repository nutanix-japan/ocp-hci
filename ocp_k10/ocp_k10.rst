.. _ocp_k10

------------------------------------------
OCP: Day 2 Backup and Restore Applications
------------------------------------------

In this section we will implement Kaster K10 as a backup and restore management applicaion.

Backup and Restore Architecture
+++++++++++++++++++++++++++++++

At the very high level, implementing backup, restore and DR involves the following components:

- Source - applications hosted on OCP cluster
- Processing/management - Kasten K10 application
- Destination - Nutanix Objects as a backup location
 
.. figure:: images/ocp_backup_design.rst 

We will also be implement Nutanix HCI snapshots to facilitate quiescing of worloads to back them up.

.. list-table::
  :widths: 25 75
  :header-rows: 1

  * - Concept
    - Storage Location
  * - Snapshots
    - Nutanix HCI Cluster (along with Storage container)
  * - Backup
    - Nutanix Objects S3 (referred to Export location)

Potential Architectures
+++++++++++++++++++++++

Architecture 1 - All in one cluster (applications, backup application and backup destination)
Architecture 2 - Source in one cluster (applications, backup application) 
 
  - Destination in second cluster (backup destination)

In this lab, we will be implementing Architecture 1 with all components in a single cluster. 

Installing VolumeSnapshotClass
+++++++++++++++++++++++++++++++

We will start by creating a VolumeSnapshotClass kubernetes object with Nutanix CSI. This helps in facilating snapshots of the source workload. 

#. In Calm go to your **Applications** > **Openshift xyz1** application

#. Go to the **Services** 

#. Select your **LB_DNS** service

#. Click on **Open Terminal**
   
   .. figure:: images/ocp_lbdns_terminal.png

   The terminal will open in a new browser tab

#. Execute the following commands in sequence
    
   .. code-block:: bash
   
    export KUBECONFIG=~/openshift/auth/kubeconfig

   .. code-block:: bash
   
    cat << EOF | oc create -f -
    apiVersion: snapshot.storage.k8s.io/v1beta1
    kind: VolumeSnapshotClass
    metadata:
      name: acs-abs-snapshot-class
      driver: csi.nutanix.com
    parameters:
      storageType: NutanixVolumes
      csi.storage.k8s.io/snapshotter-secret-name: ntnx-secret
      csi.storage.k8s.io/snapshotter-secret-namespace: kube-system
      deletionPolicy: Delete

#. 



