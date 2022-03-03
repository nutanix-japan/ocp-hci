.. _objects_prep:

Nutanix Objects as Backup Destination
+++++++++++++++++++++++++++++++++++++

In this section we will setup up Nutanix Objects to serve as a backup destination to backup our wordpress application.

Generating Access Keys
----------------------

#. Go to **Prism Central** > **Objects**

#. Note down the **ntnx-objects** object stores' public IP

#. On the top menu, click on **Access Keys**

#. Click on **+ Add people** 

#. Select **Add people not in a directory service**

#. Enter your email and name 

   .. figure:: images/objects_access_key.png

#. Click on **Next**

#. Click on **Generate Keys**

#. Once generated, click on **Download Keys**

#. Once downloaded, click on **Close**

#. Open the downloaded file to verify contents

   .. figure:: images/xyz_access_key.png

Create Bucket
-------------

We will create a bucket as backup destination

#. On the top menu, click on **Object Stores**

#. Click on **ntnx-objects**, this will open objects store management page in a separate browser tab

#. Click on **Create Bucket**

#. Enter *Initials*-k10 as the bucket name 

   .. figure:: images/create_k10_bucket.png

#. Click on **Create**

#. In the list of buckets, click on the *Initials*-k10 bucket 

   .. figure:: images/bucket_config.png

#. Click on **User Access** menu and **Edit User Access**

   .. figure:: images/bucket_ua.png

#. In the **Share Bucket xyz-k10** window, type in your email that you configured in User Access section

#. Give **Read** and **Write** permissions

   .. figure:: images/share_k10_bucket.png

#. Click on **Save**
