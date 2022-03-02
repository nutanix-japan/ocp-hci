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
      annotations:
        k10.kasten.io/is-snapshot-class: "true" # << Note the annotation to let kasten k10 use this volumesnapshotclass
    driver: csi.nutanix.com
    parameters:
      storageType: NutanixVolumes
      csi.storage.k8s.io/snapshotter-secret-name: ntnx-secret
      csi.storage.k8s.io/snapshotter-secret-namespace: ntnx-system
    deletionPolicy: Delete
    EOF


#. Run the following script to check if your OCP cluster is Kasten K10 ready

   .. code-block:: bash
    
    # Add kasten helm repo
    helm repo add kasten https://charts.kasten.io/
    # Run kasten pre-install check
    curl https://docs.kasten.io/tools/k10_primer.sh | bash

#. You would notice output as following: 

   .. code-block:: bash

      Validating Provisioners: 
      csi.nutanix.com:
        Is a CSI Provisioner  -  OK
        Storage Classes:
          nutanix-volume
            Valid Storage Class  -  OK
        Volume Snapshot Classes:
          acs-abs-snapshot-class
            Has k10.kasten.io/is-snapshot-class annotation set to true  -  OK
            Has deletionPolicy 'Delete'  -  OK

      Validate Generic Volume Snapshot:
        Pod Created successfully  -  OK
        GVS Backup command executed successfully  -  OK
        Pod deleted successfully  -  OK

      serviceaccount "k10-primer" deleted
      clusterrolebinding.rbac.authorization.k8s.io "k10-primer" deleted
      job.batch "k10primer" deleted

Installing Kasten K10 
+++++++++++++++++++++

In this section we will install Kasten K10 in our OCP cluster to backup and restore of our OCP workload (wordpress in our case). 

.. note::

   Kasten K10 can also be implemented using `OCP Operator <https://docs.kasten.io/latest/install/openshift/operator.html>`_. However, at the time of writing this lab there were issues with the Operator approach. So we will use Helm based install.

#. Install K10 using **helm** 

   Helm is pre-installed in your LB_DNS VM for your convenience.

   .. code-block:: bash

    # create a new ocp project to host kasten k10 applicaion
    oc new-project kasten-io

   .. code-block:: bash

    helm install k10 kasten/k10 --namespace=kasten-io --set scc.create=true --set route.enabled=true --set route.path="/k10" --set auth.tokenAuth.enabled=true

#. You will see output similar to the following:

   .. code-block:: bash

      NAME: k10
      LAST DEPLOYED: Tue Mar  1 06:40:27 2022
      NAMESPACE: kasten-io
      STATUS: deployed
      REVISION: 1
      TEST SUITE: None
      NOTES:
      Thank you for installing Kasten’s K10 Data Management Platform!

      Documentation can be found at https://docs.kasten.io/.

      How to access the K10 Dashboard:

      The K10 dashboard is not exposed externally. To establish a connection to it use the following `kubectl` command:

      `kubectl --namespace kasten-io port-forward service/gateway 8080:8000`

      The Kasten dashboard will be available at: `http://127.0.0.1:8080/k10/#/`

#.  Make sure all kasten k10 resources are deployed properly by running the following command:

    .. code-block:: bash
      
        oc get pods -n kasten-io

        #Output here. Make sure all pods are running

        NAME                                  READY   STATUS    RESTARTS   AGE
        aggregatedapis-svc-7874bdc658-hcvbh   1/1     Running   0          3m26s
        auth-svc-7f6f89dcc9-tcrcz             1/1     Running   0          3m26s
        catalog-svc-5c65cc87c7-bq7c8          2/2     Running   0          3m25s
        config-svc-6568f6cf-d9h8c             1/1     Running   0          3m26s
        crypto-svc-6f6bcf669b-72vr7           3/3     Running   0          3m26s
        dashboardbff-svc-7d69c79b85-q55rd     1/1     Running   0          3m26s
        executor-svc-69c8c779cc-5r5r6         2/2     Running   0          3m26s
        executor-svc-69c8c779cc-mccxp         2/2     Running   0          3m25s
        executor-svc-69c8c779cc-tpx7w         2/2     Running   0          3m25s
        frontend-svc-5cfdc5dc57-sxsqn         1/1     Running   0          3m25s
        gateway-74b76b57d-7flsm               1/1     Running   0          3m26s
        jobs-svc-6c77c5d887-pwnt5             1/1     Running   0          3m25s
        k10-grafana-8684cdd9c-jbjtc           1/1     Running   0          3m25s
        kanister-svc-74f76676c5-27t56         1/1     Running   0          3m25s
        logging-svc-7b84756c64-226rz          1/1     Running   0          3m25s
        metering-svc-5c77f69d74-b4fw4         1/1     Running   0          3m25s
        prometheus-server-85dff88779-j4rrq    2/2     Running   0          3m25s
        state-svc-84589d9df9-wkdll            2/2     Running   0          3m25s

#. Look for the route where you can access Kasten Web UI

   .. code-block:: bash

      oc get route -n kasten-io 

      #Output here.

      NAME        HOST/PORT                                     PATH    SERVICES   PORT   TERMINATION   WILDCARD
      k10-route   k10-route-kasten-io.apps.ocp1.ntnxlab.local   /k10/   gateway    http                 None

#. Make the URL by appending the path to the HOST shown in the command above

   .. code-block:: bash

    # URL + PATH
    # Example here
    # Your URL will be almost the same. Verify to make sure
    
   .. code-block:: url 

     http://k10-route-kasten-io.apps.ocp1.ntnxlab.local/k10
   
#. Copy the URL and paste in a browser in your **WindowsToolsVM**

#. You will get a login page, to get the credentials, return to the shell and run the following commands:

   Kasten K10 gives admin priveleges to k10-k10 service account during deployment. 

   This service account will have two secrets.

   We need to find the secret starting with *k10-k10-token-* and find the token value by decoding it. 

   Run the following commands to get this.

   .. code-block:: bash

    k10_login_secret=$(kubectl get serviceaccount k10-k10 -o jsonpath="{.secrets[0].name}" --namespace kasten-io)

    kubectl get secret $k10_login_secret --namespace kasten-io -ojsonpath="{.data.token}{'\n'}" | base64 --decode; echo ""

    # Sample Output here

    eyJhbGciOiJSUzI1NiIsImtpZCI6IlpDdnRDQmFvandWa0VTSWNTb042a2dpVTItVFMtd3huREpKZDM1dl9CX0kifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrYXN0ZW4taW8iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiazEwLWsxMC10b2tlbi1jdnZreCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrMTAtazEwIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZGQ2NDI4MzYtZmE4ZS00NWYyLTg2YTctNjgyMzJlMDE2NjAwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omthc3Rlbi1pbzprMTAtazEwIn0.SXo18kP4FKBEu6377n24okNg3yh8oGw2LE4JGhc4lr_V2-fm7HI13hoMnPVWaHIGPqB-NUQXtoTPQxuFO8zEjNKtFZ0g3YSfbRa_Brt-ALzclkqdVGuxPaOpmt1MDnlY6WsCkaHTAIu9pP0knEo1YTip0kxhaAJwP9v15nP3IDIcqzH4lgz28SWdOetoiMRps6bdcWfsaZxs2gLWfC5xHMd2klM8-vsDWoU2YT0WIAxrfT7As5n9b4IAmbMf80hyElypwmaRRI-q7rka-M1t4y81-TNmmd7p29wqiL04jlkuIr4oh554yQ8yfUyw0AukwUj4ARNRbfEiirH4pEs4PQ
  
#. Return to the broswer and paste this value

   .. figure:: images/ocp_k10_login_token.png

#. Click on **Sign In**

   .. note:: you may be asked to enter your company email ID and company name before you can see the Kasten K10 administration page. Please do so if necesary.

You have successfully setup Kasten backup application to backup your application.

Nutanix Objects as Kasten Destination
+++++++++++++++++++++++++++++++++++++

In this section we will setup up Nutanix Objects as a backup destination to backup our wordpress application.





