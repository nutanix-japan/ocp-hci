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

#. Make sure default is the current namespace
    
   .. code-block:: bash
   
    oc config set-context --current --namespace=default

#. Download wordpress and mysql config yaml files
   
   .. code-block:: bash

    wget https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/ocp_wordpress_deploy/mysql_deployment.yaml
    wget https://raw.githubusercontent.com/nutanix-japan/ocp-hci/main/ocp_wordpress_deploy/wordpress_deployment.yaml 

#. Make sure the nutanix-volume is the default storage class
   
   .. code-block:: bash

    oc patch storageclass nutanix-volume -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    oc get sc
    
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

#. Edit the kustomization.yaml to change your MYSQL password to a preferred password 
  
   .. code-block:: bash

    vi kustomization.yaml

#. Apply all your configuration files 

   .. code-block:: bash

    oc apply -k ./

#. Monitor pods until they are running

   .. code-block:: bash

    oc get po -w

   .. code-block:: bash

    # Sample output for the command above

    NAME                                   READY   STATUS    RESTARTS   AGE
    pod/wordpress-9c5b954c6-vdp4r          1/1     Running   0          3m11s
    pod/wordpress-mysql-77756785c8-vgtvb   1/1     Running   0          3m11s

#. Once both mysql and wordpress pods are running, we can expose the wordpress service to be able to access wordpress' Web UI.
     
   .. code-block:: bash  

    oc expose service/wordpress
   
    # Output here
    # route.route.openshift.io/wordpress exposed

   .. note:: 

    OCP exposes internal services to the outside world using the concept of routes. This routes concept existed even before the popular kubernetes Ingress object was developed.

    As a result, when you create Ingress object in latest versions of OCP, this will automatically create routes in the background.

    For more information about OCP Routes see `here. <https://docs.openshift.com/container-platform/3.11/architecture/networking/routes.html>`_

#. Get the OCP route information by running the following command

   .. code-block:: bash  

    oc get route

    # Output here

    NAME        HOST/PORT                                   PATH   SERVICES    PORT   TERMINATION   WILDCARD
    wordpress   wordpress-default.apps.xyz9.ntnxlab.local          wordpress   80                   None

#. Copy the value of **HOST/PORT** field and prepend a ``http://`` to it as we not using SSL (secure) termination

   Your route should look as follows:

   .. code-block:: url
   
    http://wordpress-default.apps.xyz9.ntnxlab.local

#. Copy the URL and paste in a browser in your **WindowsToolsVM**

#. You should see a configuration page

   .. figure:: images/ocp_wp_flash.png

#. Select **English - United States** and click on **Continue**

#. Fill in the following fields as shown in the figure below

   .. figure:: images/ocp_wp_config_options.png

#. Note the auto-generated password and copy it somewhere safe. You are also free to use your own easy to remember password.

#. You should see a installation confirmation page

   .. figure:: images/ocp_wp_install_success.png

   You will get re-directed to login page

#. Login to the wordpress site using admin and the password you used to install wordpress application

#. You will see the wordpress administraion page

#. Lets populate some data by creating a new admin user

#. Click on **Users** > **Add New**

   .. figure:: images/ocp_wp_users_addnew.png

#. Fill in the following fields as shown in the figure below

   .. figure:: images/ocp_wp_create_newuser.png

#. Click on **Add New User**

#. You will be able to see the list of users

   .. figure:: images/ocp_wp_user_list.png
  
You have succesfully set up Wordpress application with mysql backend. In the next section we will backup and restore a deleted wordpress user using Kasten K10 software.



   


   
