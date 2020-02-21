-----
Image Services
-----

*The estimated time to complete this lab is 20 minutes.*

Overview
++++++++

#. The image service feature enables you to import the images (ISOs, disk images, or any images which are supported in ESXi or Hyper-V format) directly into virtualization management.
#. The raw, vhd, vmdk, vdi, iso, qcow2 disk formats are supported.
#. You can import images from http or NFS source URL. You can use this feature to create disks for a VM from images (images that are stored in the image library or repository) and also an option to clone from an image. You must install virtIO drivers on the image prior to importing these images into the image library.
#. By creating an image of an existing bootable disk or a non-bootable disk, we are publishing it in the AOS image service across the cluster. Any Cluster Admin can use this image to spin up further virtual machines or attach it as a disk


**In this lab you will step through a creating an Image from a VM Disk Files**


Lab Setup
+++++++++

This lab requires a windows and linux VM.

Creating an Image from VM Disk
+++++++++++++++

#. We will use the Acropolis CLI for the image creation.
#. SSH to the CVM and login as “Nutanix” User.
#. Use the Image.Create from the CVM Command line

** acli image.create <Name of the new image> source_url=nfs://127.0.0.1/<Source_Container_Name/.acropolis/vmdisk/<vdisk-uuid> container=<target_container_name> image_type=kDiskImage **

#. Verify if the image exists

** nutanix@cvm$ acli image.get <image name> **

#. From the Prism GUI, create the VM and set the criteria for Type (Disk) and Operation (Clone from Image Service) and select your disk. Assign a network adapter (NIC).

Converting a Windows VM Disk files into QCOW2 format
+++++++++++++++

#. SSH to the CVM and login as “Nutanix” user.
#. For Windows VM to convert the virtual disk file to qcow2 format using the qemu-img command from the CVM command line:

** qemu-img convert -f raw nfs://127.0.0.1/Default/.acropolis/vmdisk/aeb97226-d6fc-47af-b23c-7171ed887f94 -O qcow2 nfs://127.0.0.1/Images/Win10v1903.qcow2**

#. From the Prism GUI, create the VM and set the criteria for Type (Disk) and Operation (Clone from Image Service) and select your disk. Assign a network adapter (NIC).


Converting a Linux VM Disk files into QCOW2 format
+++++++++++++++

#. SSH to the CVM and login as “Nutanix” user.
#. For Linux VM to convert the virtual disk file to qcow2 format using the qemu-img command from the CVM command line:

** qemu-img convert -f raw -c nfs://127.0.0.1/Default/.acropolis/vmdisk/aeb97226-d6fc-47af-b23c-7171ed887f94 -O qcow2 nfs://127.0.0.1/Images/Win10v1903.qcow2**

#. From the Prism GUI, create the VM and set the criteria for Type (Disk) and Operation (Clone from Image Service) and select your disk. Assign a network adapter (NIC).

Download VM Disk Files From Nutanix Container
+++++++++++++++

#. Connect to any CVM in the cluster with ‘nutanix’ user and run the following command to find the vmdisk UUID’s

** nutanix@cvm:~$ acli vm.get <VM name> include_vmdisk_paths=1 | grep -E 'disk_list|vmdisk_nfs_path|vmdisk_size|vmdisk_uuid' **

#. Sample Output
   .. figure:: images/1.png

#. Power off user VM, whose disks will be exported.
#. Using WinSCP, connect to a CVM using SFTP Protocol and port 2222 using Prism Element Credentials.
#. Enable the option to show hidden files by going to Options > Preferences > Panels and then selecting the “Show hidden files” option under the common settings
#. Navigate to the path that was found in Step 1. You can download required files from Nutanix container to your local PC now.

   .. figure:: images/2.png
