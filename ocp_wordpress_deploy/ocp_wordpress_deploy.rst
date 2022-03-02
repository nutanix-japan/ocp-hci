.. _ocp_wordpress_deploy:

-----------------------------
OCP: Deploying App with MYSQL
-----------------------------

In this section we will deploy a simple Wordpress application with MYSQL as the backend storage.

We will test backup and recovery of this application in next section.

Deploying App
+++++++++++++

#. In Calm go to your **Applications** > **Openshift xyz1** application

#. Go to the **Services** 

#. Select your **LB_DNS** service

#. Click on **Open Terminal**
   
   .. figure:: images/ocp_lbdns_terminal.png

   The terminal will open in a new browser tab

#. Execute the following commands in sequence
    
   .. code-block:: bash
   
    export KUBECONFIG=~/openshift/auth/kubeconfig

#. Create a new directory for your wordpress application and configuration files
   
   .. code-block:: bash

    mkdir wordpress 
    cd wordpress

#. Create wordpress namespace and set it as the current namespace
    
   .. code-block:: bash
   
    oc create ns wordpress
    oc config set-context --current --namespace=wordpress

#. Download wordpress and mysql config yaml files
   
   .. code-block:: bash

    wget https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/ocp_wordpress_deploy/mysql_deployment.yaml
    wget https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/ocp_wordpress_deploy/wordpress_deployment.yaml 

#. Make sure the nutanix-volume is the default storage class
   
   .. code-block:: bash

    oc patch storageclass nutanix-volume -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    oc get sc

   .. code-block:: bash
    
#. Create kustomization file to specify mysql password]

   .. code-block:: bash

    cat << EOF > kustomization.yaml
    secretGenerator:
    - name: mysql-pass
      literals:
      - password=XXXXXXX # change to your preferred MYSQL password
    resources:
    - mysql_deployment.yaml
    - wordpress_deployment.yaml
    EOF

#. Edit the kustomization.yaml to change to you preferred MYSQL password
  
   .. code-block:: bash

    vi kustomization.yaml

#. Apply all your configuration files 

   .. code-block:: bash

    oc apply -k ./
     
Create OCP Route 
++++++++++++++++

We will create a OCP Route to access the Wordpress application.

