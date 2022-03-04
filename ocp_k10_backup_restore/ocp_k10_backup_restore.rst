.. _ocp_k10_backup_restore:

In this section we will backup and restore Wordpress applicaion using Kasten K10.

-------------------------------
Backup and Restore Applications
-------------------------------

Backup Wordpress
++++++++++++++++

#. Go to Kasten dashboard

#. Click on **Applications** 

   .. figure:: images/kasten_apps.png

#. Click on **0 Compliant** button

#. As we installed our Wordpress application in the ``default`` namespace, click on **Create a Policy** as shown here

   .. figure:: images/kasten_default_policy.png

#. In the **New Policy** window, leave the policy name as **default-backup**

#. Select on **Enable Backups via Snapshot Exports** 

#. Make sure **Export Location Profile** is point to **ntnx-object** location profile we created before

#. Leave everything else as is and click on **Create Policy** at the bottom of the window.

   .. figure:: images/kasten_default_policy_options.png

#. Under **default-backup** policy, click on **Run Once**

   .. figure:: images/kasten_policy_run_once.png

#. Confirm by clicking on **Run Policy**

#. Return to Kasten dashboard

#. Under **Actions**, click on the **Policy Run** for **default-backup** policy

   .. figure:: images/actions_policy_run.png

#. You will see the details as follows (it will take a few minutes for the backup to run)

   .. figure:: images/kasten_default_backup_success.png

#. Once the backup is finished, you will see that the default namespace/application is Compliant under **Dashboard** > **Applications**

   .. figure:: images/kasten_compliant_defaut_ns.png

#. You can go to Prism Central > Tasks to see snapshots tasks requested by Kasten 

   .. figure:: images/pc_snapshot_tasks.png

#. The volumesnapshot object we created along with other Objects in Nutanix CSI provider has enabled us to do this snapshot and backup

You have completed backup, now let us move on to simulating data loss and restore operations.

Simulating Data Loss 
++++++++++++++++++++

#. Go to the Wordpress applications Web UI in the other browser tab.

   .. note::

      If you accidentally closed the tab, you can do the following to get the URL
      
      Follow instruction here to get the Wordpress Web URL :ref:`ocp_wordpress_deploy`

#. Click on **Users** menu 

#. Hover your mouse prompt over the **admin_1** user

#. You will see the Delete menu

#. Click on **Delete**
   
   .. figure:: images/wordpress_user_delete.png

#. Click on **Confirm Deletion**

Restoring Worpress Application
++++++++++++++++++++++++++++++

#. Browse to your Kasten dashboard

#. Click on the **1 Compliant** part in Applications

   .. figure:: images/kasten_restore_start.png

#. Click on restore 

   .. figure:: images/kasten_restore_2.png

#. You will see two restore points and be asked to **Select an Instance**
  
   - Restore from backup (this will restore from Nutanix DFS snapshot)
   - Restore from export (this will restore from Objects stores xyz-k10 bucket)

#. Select the EXPORTED one to ntnx-objects

   .. figure:: images/kasten_select_objects_restore.png

#. Click on **Deselect All Artifacts**

#. Select only the PVC and Deployments as shown here
   
   - **PVC** - mysql-pv-claim
   - **PVC** - wp-pv-claim 
   - **Deployments** - wordpress
   - **Deployments** - wordpress-mysql
 
   .. figure:: images/kasten_restore_artifacts.png
   
#. Click on **Restore** and **Restore** again in the confirmation window ]

#. Return to the Kasten Dashboard to monitor the progress 

#. Go to your ssh shell and execute the following command to observe the pods

   .. code-block:: bash

    oc get po -n default -w

    # Output here
    # The wordpress and mysql pod will be terminated
    # Some restore operation pods will get created
    # Data will be restored
    # Wordpress and mysql pod will be created

    NAME                               READY   STATUS    RESTARTS   AGE
    wordpress-9c5b954c6-bpblk          1/1     Running   0          5h27m
    wordpress-mysql-77756785c8-r4b5r   1/1     Running   0          5h27m
    wordpress-mysql-77756785c8-r4b5r   1/1     Terminating   0          5h28m
    wordpress-9c5b954c6-bpblk          1/1     Terminating   0          5h28m
    wordpress-9c5b954c6-bpblk          0/1     Terminating   0          5h28m
    wordpress-mysql-77756785c8-r4b5r   0/1     Terminating   0          5h28m
    wordpress-9c5b954c6-bpblk          0/1     Terminating   0          5h28m
    wordpress-9c5b954c6-bpblk          0/1     Terminating   0          5h28m
    wordpress-mysql-77756785c8-r4b5r   0/1     Terminating   0          5h28m
    wordpress-mysql-77756785c8-r4b5r   0/1     Terminating   0          5h28m
    affinity-pod-1                     0/1     Pending       0          0s
    affinity-pod-1                     0/1     Pending       0          0s
    affinity-pod-0                     0/1     Pending       0          0s
    affinity-pod-0                     0/1     Pending       0          0s
    affinity-pod-1                     0/1     Pending       0          8s
    affinity-pod-0                     0/1     Pending       0          8s
    affinity-pod-1                     0/1     ContainerCreating   0          8s
    affinity-pod-0                     0/1     ContainerCreating   0          8s
    affinity-pod-0                     0/1     ContainerCreating   0          19s
    affinity-pod-1                     0/1     ContainerCreating   0          20s
    affinity-pod-0                     1/1     Running             0          20s
    affinity-pod-1                     1/1     Running             0          21s
    affinity-pod-1                     1/1     Terminating         0          25s
    affinity-pod-0                     1/1     Terminating         0          25s
    restore-data-6p96k                 0/1     Pending             0          0s
    restore-data-6p96k                 0/1     Pending             0          0s
    restore-data-6p96k                 0/1     ContainerCreating   0          0s
    restore-data-4887h                 0/1     Pending             0          0s
    restore-data-4887h                 0/1     Pending             0          0s
    restore-data-4887h                 0/1     ContainerCreating   0          1s
    restore-data-6p96k                 0/1     ContainerCreating   0          3s
    restore-data-6p96k                 1/1     Running             0          4s
    restore-data-6p96k                 1/1     Terminating         0          8s
    restore-data-4887h                 0/1     ContainerCreating   0          8s
    restore-data-4887h                 1/1     Running             0          10s
    restore-data-4887h                 1/1     Terminating         0          16s
    wordpress-mysql-77756785c8-d4djd   0/1     Pending             0          0s
    wordpress-9c5b954c6-qmq86          0/1     Pending             0          0s
    wordpress-mysql-77756785c8-d4djd   0/1     Pending             0          0s
    wordpress-9c5b954c6-qmq86          0/1     Pending             0          0s
    wordpress-mysql-77756785c8-d4djd   0/1     ContainerCreating   0          0s
    wordpress-9c5b954c6-qmq86          0/1     ContainerCreating   0          0s
    wordpress-mysql-77756785c8-d4djd   0/1     ContainerCreating   0          3s
    wordpress-9c5b954c6-qmq86          0/1     ContainerCreating   0          7s
    affinity-pod-0                     0/1     Terminating         0          59s
    wordpress-mysql-77756785c8-d4djd   1/1     Running             0          10s #<< restored wordpress
    affinity-pod-1                     0/1     Terminating         0          59s
    wordpress-9c5b954c6-qmq86          1/1     Running             0          11s #<< restored mysql
    affinity-pod-1                     0/1     Terminating         0          63s
    affinity-pod-1                     0/1     Terminating         0          63s
    affinity-pod-0                     0/1     Terminating         0          63s
    affinity-pod-0                     0/1     Terminating         0          63s
    restore-data-6p96k                 0/1     Terminating         0          40s
    restore-data-6p96k                 0/1     Terminating         0          47s
    restore-data-6p96k                 0/1     Terminating         0          47s
    restore-data-4887h                 0/1     Terminating         0          48s

#. Go back to Kasten Wed UI and you will see restore completing successfully

   .. figure:: images/kasten_restore_success.png

#. Login to Wordpress GUI to check if the deleted user is now present

   .. figure:: images/wordpress_restored_user_state.png

You have succesfully restored the lost account. 

Takeaways
+++++++++

- Applications hosted on OCP on Nutanix can be backed up to Nutanix Objects/Files
- Nutanix provides Infrastructure for OCP workloads 
- Nutanix provides Objects/Files storage for backup workloads

