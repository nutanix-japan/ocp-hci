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

#. Create a kustomization yaml file in your app Directory
   
   .. code-block:: bash

    mkdir wordpress 
    cd wordpress

    # Set default namespace as current context

    oc config set-context --current --namespace=default

    # Download wordpress and mysql config yaml files
    wget 
    wget 
     
