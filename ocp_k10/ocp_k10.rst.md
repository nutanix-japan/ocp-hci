---
title: Deploying Backup Application
---

In this section we will implement Kasten K10 as a backup and restore
management applicaion.

# Backup and Restore Architecture

At the very high level, implementing backup, restore and DR involves the
following components:

-   **Source** - applications hosted on OCP cluster
-   **Processing/management** - Kasten K10 application
-   **Destination** - Nutanix Objects as a backup location

We will also be implement Nutanix HCI snapshots to facilitate quiescing
of worloads to back them up. This will be done using VolumeSnapshotClass
kubernetes object in this lab.

  -----------------------------------------------------------------------
  Concept             Storage Location                       RTO/RPO
  ------------------- -------------------------------------- ------------
  Backup (Snapshots)  Nutanix HCI Cluster (along with        High
                      Storage container)                     

  Exports             Nutanix Objects S3 (stored in backup   Medium
                      target)                                
  -----------------------------------------------------------------------

# Potential Architectures

-   Architecture 1 - All in one cluster (applications, backup
    application and backup destination)

    -   Workload snapshots and backup exports are stored in the same
        Nutanix HCI cluster
    -   Snapshots are provided by Nutanix DFS

    ![](images/backup_design_1.png)

-   Architecture 2 - Source in one cluster (applications, backup
    application)

    -   Workload snapshot location is the source Nutanix HCI cluster
    -   Snapshots are provided by Nutanix DFS
    -   Backup export location in the Objects/Files cluster

    ![](images/backup_design_2.png)

In this lab, we will be implementing Architecture 1 with all components
in a single cluster.

# Installing VolumeSnapshotClass

Before you proceed with this section, verify that Nutanix CSI operator
version and make sure it is `2.5.1` - we need this version to be able to
implement volumesnapshotclass

![](images/csi_operator_version.png)

We will start by creating a VolumeSnapshotClass kubernetes object with
Nutanix CSI. This helps in facilitating snapshots of the source
workload.

1.  In Calm go to your **Applications** \> **Openshift xyz1**
    application

2.  Go to the **Services**

3.  Select your **LB_DNS** service

4.  Click on **Open Terminal**

    ![](images/ocp_lbdns_terminal.png)

    The terminal will open in a new browser tab

5.  Execute the following commands in sequence

    ``` bash
    export KUBECONFIG=~/openshift/auth/kubeconfig
    ```

    ``` bash
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
    ```

6.  Run the following script to check if your OCP cluster is Kasten K10
    ready

    ``` bash
    # Add kasten helm repo
    helm repo add kasten https://charts.kasten.io/
    # Run kasten pre-install check
    curl https://docs.kasten.io/tools/k10_primer.sh | bash
    ```

7.  You would notice output as following:

    ``` bash
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
    ```

8.  Make sure all pre-checks are OK. If not, go back and check the
    associated errors and resolve them. Ask your instructor for help.

# Installing Kasten K10

In this section we will install Kasten K10 in our OCP cluster to backup
and restore of our OCP workload (wordpress in our case).

::: note
::: title
Note
:::

Kasten K10 can also be implemented using [OCP
Operator](https://docs.kasten.io/latest/install/openshift/operator.html).
However, at the time of writing this lab there were issues with the
Operator approach. So we will use Helm based install.
:::

1.  Install K10 using **helm**

    Helm is pre-installed in your LB_DNS VM for your convenience.

    ``` bash
    # create a new ocp project to host kasten k10 applicaion
    oc new-project kasten-io
    ```

    ``` bash
    helm install k10 kasten/k10 --namespace=kasten-io --set scc.create=true --set route.enabled=true --set route.path="/k10" --set auth.tokenAuth.enabled=true
    ```

2.  You will see output similar to the following:

    ``` bash
    NAME: k10
    LAST DEPLOYED: Tue Mar  1 06:40:27 2022
    NAMESPACE: kasten-io
    STATUS: deployed
    REVISION: 1
    TEST SUITE: None
    NOTES:
    Thank you for installing Kasten’s K10 Data Management Platform!

    Documentation can be found at https://docs.kasten.io/
    ```

3.  Make sure all kasten k10 resources are deployed properly by running
    the following command:

    ``` bash
    oc get pods -n kasten-io -w

    # Output here. Wait until  all pods are running
    # This may take a few minutes 

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
    ```

4.  Look for the route where you can access Kasten Web UI

    ``` bash
    oc get route -n kasten-io 

    #Output here.

    NAME        HOST/PORT                                     PATH    SERVICES   PORT   TERMINATION   WILDCARD
    k10-route   k10-route-kasten-io.apps.ocp1.ntnxlab.local   /k10/   gateway    http                 None
    ```

5.  Make the URL by appending the path to the HOST shown in the command
    above

    ``` bash
    # URL + PATH
    # Example below
    # Your URL will be almost the same. Verify to make sure
    ```

    ``` url
    http://k10-route-kasten-io.apps.ocp1.ntnxlab.local/k10/#

    # if the above URL doesn't work try this one:

    http://k10-route-kasten-io.apps.ocp1.ntnxlab.local/k10/
    ```

6.  Copy the URL and paste in a browser in your **WindowsToolsVM**

7.  You will get a login page, to get the credentials, return to the
    shell and run the following commands:

    Kasten K10 gives admin priveleges to k10-k10 service account during
    deployment.

    This service account will have two secrets.

    We need to find the secret starting with *k10-k10-token-* and find
    the token value by decoding it.

    Run the following commands to get this:

    ``` bash
    k10_login_secret=$(kubectl get serviceaccount k10-k10 -o jsonpath="{.secrets[0].name}" --namespace kasten-io)

    kubectl get secret $k10_login_secret --namespace kasten-io -ojsonpath="{.data.token}{'\n'}" | base64 --decode; echo ""
    ```

    ``` bash
    # Sample output shown here - your token will be different

    eyJhbGciOiJSUzI1NiIsImtpZCI6IlpDdnRDQmFvandWa0VTSWNTb042a2dpVTItVFMtd3huREpKZDM1dl9CX0kifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrYXN0ZW4taW8iLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiazEwLWsxMC10b2tlbi1jdnZreCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJrMTAtazEwIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiZGQ2NDI4MzYtZmE4ZS00NWYyLTg2YTctNjgyMzJlMDE2NjAwIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omthc3Rlbi1pbzprMTAtazEwIn0.SXo18kP4FKBEu6377n24okNg3yh8oGw2LE4JGhc4lr_V2-fm7HI13hoMnPVWaHIGPqB-NUQXtoTPQxuFO8zEjNKtFZ0g3YSfbRa_Brt-ALzclkqdVGuxPaOpmt1MDnlY6WsCkaHTAIu9pP0knEo1YTip0kxhaAJwP9v15nP3IDIcqzH4lgz28SWdOetoiMRps6bdcWfsaZxs2gLWfC5xHMd2klM8-vsDWoU2YT0WIAxrfT7As5n9b4IAmbMf80hyElypwmaRRI-q7rka-M1t4y81-TNmmd7p29wqiL04jlkuIr4oh554yQ8yfUyw0AukwUj4ARNRbfEiirH4pEs4PQ

    # Copy the token from your ssh console and paste in the Kasten login UI
    ```

    ::: note
    ::: title
    Note
    :::

    If the secret doesn\'t work: use the following commands to get the
    second secret in serviceaccount k10-k10

    ``` bash
    k10_login_secret=$(kubectl get serviceaccount k10-k10 -o jsonpath="{.secrets[1].name}" --namespace kasten-io)

    kubectl get secret $k10_login_secret --namespace kasten-io -ojsonpath="{.data.token}{'\n'}" | base64 --decode; echo ""
    ```
    :::

8.  Return to the broswer and paste the secret value

    ![](images/ocp_k10_login_token.png)

9.  Click on **Sign In**

    ::: note
    ::: title
    Note
    :::

    you may be asked to enter your company email ID and company name
    before you can see the Kasten K10 administration page. Please do so
    if necesary.
    :::

# Setup up Backup Target

We will create a backup target to point to our bucket we created in the
previous section `objects_prep`{.interpreted-text role="ref"}. In Kasten
this is called Location Profiles.

1.  In Kasten dashboard click on **Settings**

    ![](images/kasten_dash.png)

2.  Click on **Locations** \> **+ New Profile**

    ![](images/kasten_location_profile.png)

3.  Enter the following details

    -   **Profile Name** - ntnx-objects
    -   **Cloud Storage Provider** - S3 Compatible
    -   **S3 Access Key** - Access key from the file you downloaded in
        the previous section
    -   **S3 Secret Key** - Secret key from the file you downloaded in
        the previous section
    -   **Endpoint** - Public IP of ntnx-objects Object Store (e.g.
        <https://10.42.32.18>)
    -   **Skip certificate chain and hostname verification** - Checked
        and confirm to **Disable SSL Verify** in the prompt
    -   **Region** - Leave blank
    -   **Bucket Name** - *Initials*-k10 (e.g. xyz-k10)

    ![](images/location_profile_config.png)

4.  Click on **Save**

5.  Kasten will do a validation and the profile will be available in the
    **Settings** \> **Locations** page

    ![](images/location_profile.png)

    ::: note
    ::: title
    Note
    :::

    If there are validation errors, please go back to checking all the
    input parameters for creating location profile
    :::

You have successfully setup Kasten backup application to backup your
application.
