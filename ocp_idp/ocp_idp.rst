.. _ocp_zk_deploy:

-------------------
OCP: Deploying IDP
-------------------

In this lab we will deploy a IDP connection to your OCP cluster to provide Authentication and Authorization.

We will use the AutoAD Active Directory VM as the IDP as it is already installed in your HPOC/SPOC cluster.

#. In Calm go to your **Applications** > **Openshift ocp1** application

#. Go to the **Services** 

#. Select your **LB_DNS** service

#. Click on **Open Terminal**
   
   .. figure:: images/ocp_lbdns_terminal.png

   The terminal will open in a new browser tab

#. Execute the following commands in sequence
    
   .. code-block:: bash
   
    export KUBECONFIG=~/openshift/auth/kubeconfig

   .. code-block:: bash

    # Create a kubernets secret for AutoAS AD administrator password
    oc create secret generic ldap-secret --from-literal=bindPassword='nutanix/4u' -n openshift-config

#. Setup the OAuth provider
   
   .. code-block:: bash

    cat << EOF | oc create -f -
    apiVersion: config.openshift.io/v1
    kind: OAuth
    metadata:
      name: cluster
    spec:
        identityProviders:
            - name: ntnxlab.local 
            mappingMethod: claim 
            type: LDAP
            ldap:
                attributes:
                id: 
                - sAMAccountName
                email: []
                name: 
                - displayName
                preferredUsername: 
                - sAMAccountName
                bindDN: administrator@ntnxlab.local 
                bindPassword: 
                    name: ldap-secret
                insecure: true
                url: ldap://dc.ntnxlab.local/CN=Users,DC=ntnxlab,DC=local?sAMAccountName
    EOF

#. Setup the LDAP sync

   .. code-block:: bash

        cat << EOF | oc appl
        kind: LDAPSyncConfig
        apiVersion: v1
        url: ldap://dc.ntnxlab.local:389
        bindDN: administrator@ntnxlab.local 
        bindPassword: nutanix/4u
        insecure: true
        groupUIDNameMapping:
        CN=SSP Admins,CN=Users,DC=ntnxlab,DC=local: OCP_SSP_Admins
        CN=Domain Admins,CN=Users,DC=ntnxlab,DC=local: OCP_Cluster_Admins
        augmentedActiveDirectory:
        groupsQuery:
            baseDN: CN=users,DC=ntnxlab,DC=local
            scope: sub
            derefAliases: never
            pageSize: 0
        groupUIDAttribute: dn
        groupNameAttributes: [ cn ]
        usersQuery:
            baseDN: cn=users,dc=ntnxlab,dc=local
            scope: sub
            derefAliases: never
            filter: (objectclass=person)
            pageSize: 0
        userNameAttributes: [ sAMAccountName ] 
        groupMembershipAttributes: [ memberOf ]

    
     
