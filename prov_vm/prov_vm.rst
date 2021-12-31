.. _prov_vm:


Deploy Provisioning VM 
++++++++++++++++++++++++++

Configure Provisioning VM Blueprint
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now lets configure and deploy Provisioning VM

#. In **Prism Central** > **Services** > **Calm**
 
#. Click on **Blueprints**

#. Select your Provisioning VM ``OCP_Prov_VM`` blueprint

We need to configure network and credentials for the blueprint so it can be deployed in your HPOC/SPOC.

#. Select the Provisioning_VM service 

#. In the services' VM properties, Choose Primary as the NIC 1

   .. figure:: images/ocp_prov_vm_nic1.png

#. Click on **Save**
   
   .. figure:: images/ocp_bp_save.png

#. Click on **Credentials** (next to Save)

#. Copy and paste the following primary key (this could be any primary key that you may already have)
   
   .. code-block:: bash

    -----BEGIN OPENSSH PRIVATE KEY-----
    b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
    NhAAAAAwEAAQAAAQEA5//EaGHrEDLtng7U252IFsWvglORQXCGCpY3urfMOjl/LMY+41s7
    ug028XPlMO+kw5J6ylIgcepWijrfyqBTklHjXVNWo0WuukjrbEwG2CNHwdaCAGwBXhzNHV
    kgiaMj9mCCoNhZ6UZW+X2WsdpSjgkTKKXGoLh6R5wm+5ZdXvsjlZcwrNvuwKJ8uAsdWafV
    KlafThHUNHpLh+ZvNiinLGmfiBx1DTZzIUEsjf4fYcRk1drUbJnrMO6eXHPa6aPjx/MO9k
    eF6VLOM5dqZrdAYedMU3MFxWuTyXB6o6kd4ddmd/twSTC7cgRcADtaE5zcB3ZNmz2Ek5Lh
    dkIm9Yu8HwAAA9AATH8HAEx/BwAAAAdzc2gtcnNhAAABAQDn/8RoYesQMu2eDtTbnYgWxa
    +CU5FBcIYKlje6t8w6OX8sxj7jWzu6DTbxc+Uw76TDknrKUiBx6laKOt/KoFOSUeNdU1aj
    Ra66SOtsTAbYI0fB1oIAbAFeHM0dWSCJoyP2YIKg2FnpRlb5fZax2lKOCRMopcaguHpHnC
    b7ll1e+yOVlzCs2+7Aony4Cx1Zp9UqVp9OEdQ0ekuH5m82KKcsaZ+IHHUNNnMhQSyN/h9h
    xGTV2tRsmesw7p5cc9rpo+PH8w72R4XpUs4zl2pmt0Bh50xTcwXFa5PJcHqjqR3h12Z3+3
    BJMLtyBFwAO1oTnNwHdk2bPYSTkuF2Qib1i7wfAAAAAwEAAQAAAQEA59REnw4iB+jsW4J0
    oBSWFjNrA6+gOiVIPLIJdHMQCKrF86cS1rynPW7RFYx7XWBLZAcu3pgcxdimq7DTSrbVbC
    SaL/sEW1lZldsiLfD28eqnXQKOm/9Rior7SLV0dzVnzV55imNHzXy1ktA10vnYfwiAdaDa
    kGZoAsD+j2+dVDd3+qCLftw/uvXrlp8gx+bvX8xcoMxsckdKIQUE8zZJedV3QBfCHSnagc
    ub3BI3x5zaYTx3lZqaYYknkDr9gyGHuvPywJTMUsoAXhvt3/AONByLOUwpNt92svR+ljwy
    wszP7XqLtZEFA/Xb0t8fHd2ZhtndR9V5wTz6fPu/xvO6MQAAAIBQU03ZxyZyt72FygB/n+
    hBnVru7B7e3m2EjaLW6HKWmUaHez3go2GjAfQFMXHtHKkUbZ/25JwFNIYPRqgan45sgrxl
    UYGPCcwYMMqAg+8YsL8sxl6Ypc5YyoSUZTjRScGZw8us5qV/TBaXiY0TK3dqNM0VI7LFj6
    t89iKi6/KeZQAAAIEA+y0rIun5o27XfB+S4biP8xV1P8dls8NFg1uwFs50dJQfSozokW8z
    lgY92LvjHzgVs8OCj2HfrcM3+ySj6sZgRHkr+eL0fdum5ttYcuJx6391nuE4FehS5jtyjG
    3FIjVW9B5PPx7MXucp9w9ewSVNFo7Fn41d6++ALFThF1c2jucAAACBAOx0UbbNg4BTUKGU
    e8jg/uY36BFUJ8znKrPqLxMZv6YCW90dozPY0bFoV2iaPaqtKDJi/zrfyiteQkPA5o2MCt
    EEa8CaSUJaybyqTLwPfPX02U8jEwUF8n/5NGusMer8cIy0OFe1WyVEuaUk6zWgF7HIn/ba
    impp30n2vf7rkNoJAAAAFGxha3NobWlAQzAyRjcwRkxNTDg1AQIDBAUG
    -----END OPENSSH PRIVATE KEY-----

#. Click on **Save**

   .. figure:: images/ocp_bp_save_cred.png

#. Click on **Back** to return to the blueprints main window

#. You will notice a few warnings for KUBEADMIN and KUBECONFIG variables. These can be ignored as they will be auto-generated.

   .. figure:: images/ocp_bp_warnings.png


Launch Provisioning VM Blueprint
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now it is time to launch the provisioning VM blueprint.

#. Click on **Launch** button

#. Give the application a name *Initials*_Prov_VM

   .. figure:: images/ocp_prov_vm_bp_launch.png

#. Click on **Deploy**

#. Go to Audit and monitor the deployment tasks. There will be detailed logging for all tasks

#. This should take about 10-15 minutes

#. Once deployed the Calm application will be in a running state

   .. figure:: images/ocp_prov_vm_audit.png

#. **Optional step** - You are also able to ssh into the provisioning VM using the application's (if you are curious to see the downloaded files for OCP setup) > **Services** > **Open Terminal**
   
   .. figure:: images/ocp_prov_vm_ssh.png

   .. code-block:: zsh 

        [core@Openshift-provisioning-0-211225-210356 ~]$ ls -lRh openshift/
        openshift/:
        total 1.6G
        -rw-rw-r--. 1 core   core   7.3M Dec 26 05:11 coreos-installer
        -rwxr-xr-x. 2 core   core   118M Nov  4 19:41 kubectl
        -rwxr-xr-x. 2 core   core   118M Nov  4 19:41 oc
        -rwxr-xr-x. 1 core   core   369M Nov 22 17:12 openshift-install
        -rw-r--r--. 1 core   core    954 Nov  4 19:41 README.md
        -rw-rw-r--. 1 core   core   988M Dec 26 05:12 rhcos-live.x86_64.iso ## << this is RHOCS ISO
        drwxrwxr-x. 2 apache apache  105 Dec 26 05:12 web

        openshift/web:
        total 144M
        -rw-rw-r--. 1 apache apache 7.3M Dec 26 05:12 coreos-installer 
        -rw-rw-r--. 1 apache apache  48M Dec 26 05:11 openshift-client-linux.tar.gz ## << this is OCP Client
        -rw-rw-r--. 1 apache apache  89M Dec 26 05:11 openshift-install-linux.tar.gz ## << this is OCP Server

Now we have the provisioning VM up and running. This section of the lab is done.

.. figure:: images/ocp_lab_status_1.png

We will proceed to deploy a OCP cluster in the next section of the lab. 








