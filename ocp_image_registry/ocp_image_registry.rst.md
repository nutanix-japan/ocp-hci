---
title: "OCP: Provision Storage for Image Registry"
---

OCP comes with a in-built container image registry. The advantages of
these container image registry are the following:

-   Integrates with OCP clusters authentication and authorization system
-   Provides a image source local to the cluster for all running
    workloads
-   Integrates with OCP clusters CICD workflows - for example when a new
    cotainer images is uploaded to this registry, the cluster can update
    the image of all the running running contianers
-   Registry can be easily scaled up or down
-   Image data is stored in a cloud storage

In this lab we will create a persistent volume claim (PVC) and present
it as storage to the running image registry on this OCP cluster. This
will be our first use case for Nutanix provided HCI storage.

The PVC will be 100 GB.

# Creating the PVC

1.  In Calm go to your **Applications** \> **Openshift ocp1**
    application

2.  Go to the **Services**

3.  Select your **LB_DNS** service

4.  Click on **Open Terminal**

    ![](images/ocp_lbdns_terminal.png)

    The terminal will open in a new browser tab

5.  Export the OCP cluster\'s KUBECONFIG file to environment so we can
    perform `oc` commands (if you haven\'t already)

    ``` bash
    export KUBECONFIG=~/openshift/auth/kubeconfig
    ```

6.  Copy the following StorageClass configuration script and paste it in
    the command line window

    ``` bash
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
    ```

    ``` bash
    # example output here for the above command
    # persistentvolumeclaim/image-registry-claim created
    ```

7.  Confirm the creation of pvc using the following command

    ``` bash
    oc get pvc -n openshift-image-registry
    ```

    ``` bash
    # example output here for the above command
    # NAME                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS     AGE
    # image-registry-claim   Bound    pvc-e31a2dc7-fcb0-498b-bb88-ffdf61aeaa81   100Gi      RWO            nutanix-volume   2m14s
    ```

Presenting the PVC to OCP Image Registry
+++++++++++++++++++++++++++++++++++++

1.  Copy the following patch configuration script and paste it in the
    command line window

    ``` bash
    oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"managementState":"Managed","storage":{"pvc":{"claim":"image-registry-claim"}},"rolloutStrategy": "Recreate"}}'
    ```

    ``` bash
    # example output here for the above command
    # config.imageregistry.operator.openshift.io/cluster patched
    ```

    This will patch the image registry with the created storage (PVC) by
    re-creating imageregistry operator.

2.  To see the changes at the operator level execute the following
    command:

    ``` bash
    oc describe configs.imageregistry.operator.openshift.io
    ```

    ``` bash
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
    ```

::: note
::: title
Note
:::

```{=html}
<body><font color="green">Validation Task 9 - Take a screenshot of the of your image-registry-claim from the above command</font></body>
```
:::

To check the existence this PV in Prism Element, check the volume detail
in Storage container.

1.  Go to your **Prism Element** \> **Storage** \> **Volume Group**

2.  You should see a 100 GB disk provisioned as shown below

    ![](images/ocp_pv_vg.png)

You have successfully created a storage PVC in Nutanix HCI and presented
it to a resource in OCP cluster. We will create other resources and
present Nutanix HCI storage to them in the subsequent sections of the
lab.
