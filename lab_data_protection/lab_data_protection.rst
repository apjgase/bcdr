.. _lab_data_protection:

---------------------
Data Protection Lab
---------------------

Overview
++++++++

Nutanix provides the ability to perform VM/vDisk-level storage snapshots. Protection Domains (PDs) are the construct for grouping VMs and applying snapshot and replication policies.

In this exercise you will use Prism to create and restore from VM snapshots, as well as create a Protection Domain for your VMs.

Data Protection
+++++++++++++++

VM Snapshots
............

#. In **Prism Element > VM > Table**, select your *Initials*-**Linux_VM** VM.

#. If the VM is powered on, perform a **Guest Shutdown** power action.

#. Select the VM and click **Take Snapshot** from the menu below the table.

#. Provide a name for your snapshot and click **Submit**.

#. Select the **VM Snapshots** tab below the table to view the available snapshots for the selected VM.

   .. figure:: images/manage_workloads_04.png

#. Under **Actions**, click **Details** to see all of the VM’s properties at the time of the snapshot.

   You can see the snapshot contains VM state in addition to just its data.

   *Now it's time to break your VM!*

#. Click **Update** to modify your VM and remove both the CD-ROM and DISK by clicking the **X** icon for each item.

#. Click **Save**.

#. Attempt to power on the VM and launch its console window.

   Note that the VM no longer has any disks from which to boot and that the 2048 game is displayed.

#. Power off the VM.

#. Under **VM Snapshots**, select your snapshot and click **Restore** to revert the VM to a functioning state.

   Alternatively you can click **Clone** to restore to a new VM.

#. Verify that the VM boots successfully.

As previously mentioned, Nutanix snapshots use a `redirect-on-write <https://nutanixbible.com/#anchor-book-of-acropolis-snapshots-and-clones>`_ approach that does not suffer from the performance degradation of chained snapshots found in other hypervisors.

Protection Domains
..................

#. In **Prism Element > Data Protection > Table**, click **+ Protection Domain > Async DR** to begin creating a PD.

#. When opening the Data Protection context of the menu a warning screen will appear. Click on the **OK** button to move forward.

 .. figure:: images/data_protection_01.png

#. Provide a name for the PD, and click **Create**.

#. Filter or scroll to select the VMs created during this lab that you want to add to the PD.

#. Click **Protect Selected Entities** and verify the VMs appear under **Protected Entities**.

   Consistency groups allow you to group multiple VMs to be snapshot at the same time, e.g. multiple VMs belonging to the same application.

   .. note:: Nutanix snapshots can perform application consistent snapshots for supported operating systems with NGT installed. Each VM using application consistent snapshots will be part of its own consistency group.

#. Click **Next**.

#. Click **New Schedule** to define Recovery Point Objective (RPO) and retention.

#. Configure your desired snapshot frequency (e.g. Repeat every 1 hour)

   .. note::

      AHV supports NearSync snapshots, with RPOs as low as 1 minute.

   .. note::

      Multiple schedules can be applied to the same PD, allowing you to take and retain X number of hourly, daily, monthly snapshots.

#. Configure a retention policy (e.g. Keep the last 5 snapshots)

   .. note::

      For environments with remote cluster(s) configured, setting up replication is as easy as defining how many snapshots to keep at each remote site.

      .. figure:: images/snapshot_02.png

#. Click **Create Schedule**.

#. Click **Close** to exit.

Additional information can be found `here <https://nutanixbible.com/#anchor-book-of-acropolis-backup-and-disaster-recovery>`_.

That's it! You've successfully configured native data protection in Prism.


Planned failover
+++++++++++++++

#. Log in to your VM and create a text file with a current date and time

#. In **Prism Element > Data Protection > Table**, right-click on your protection domain and select **Take snapshot** 

    .. figure:: images/takeasnap.png

#. Select both local and remote sites and click **Save**
 
#. Click replications, scroll down and check last successful or ongoing replication

#. Select the target protection domain and click the Migrate action link. The Migrate Protection Domain dialog box appears. Select the site where you want to migrate the protection domain. The VMs that are part of the protection domain, but cannot be recovered on the remote site are also displayed. 

    .. figure:: images/migrate.png

   .. note::

    Migrating a protection domain does the following:
    Creates and replicates a snapshot of the protection domain.
    Powers off the VMs on the local site.
    Note: The data protection service waits for 5 minutes for the VM to shut down. If the VM does not get shutdown within 5 minutes, it is automatically powered off.
    Creates and replicates another snapshot of the protection domain.
    Unregisters all VMs and volume groups and removes their associated files.
    Marks the local site protection domain as inactive.
    Restores all VM and volume group files from the last snapshot and registers them with the same UUIDs at the remote site.
    Note: If you are migrating a protection domain to a different hypervisor (for example, ESXi to AHV), all VM and volume group files register with a new UUID at the recovery site.
    Marks the remote site protection domain as active.
    
#. Once migration will be completed, login to remote site

#. In **Prism Element > Data Protection > Table**, select your protection domain

#. Check incoming transfers -> last successful, match the snapshot ID with primary site

#. Check protection domain details - protection domain mode should be "active" 

    .. figure:: images/active.png

#. Power on your VM at the DR site, login and check your text file. Add one more line with the current date and time

Performing Failback
..................

#. Login to the Web console of the DR site (the site where the protection domain is currently active).

#. From the Async DR tab under Data Protection, select the Protection Domain that you want to failback.

#. Click **Migrate**. The Migrate Protection Domain dialog box appears. 

#. Select the site where you want to migrate the Protection Domain. The VMs that are part of the Protection Domain, but cannot be recovered on the remote site are also displayed. 

#. When the field entries are correct, click the **Save** button.

#. Once migration will be completed, login to primary site

#. In **Prism Element > Data Protection > Table**, select your protection domain

#. Check protection domain details - protection domain mode should be "active" 

    .. figure:: images/active.png

#. Power on your VM at the primary site, log in and check your text file. Add one more line with the current date and time.


Unplanned failover
+++++++++++++++

#. At the primary site, in **Prism Element > Data Protection > Table**, right-click on your protection domain and select **Take snapshot** 

#. Once transfer would be completed, notify an instructor 

#. Nutanix instructor will simulate site A failure

#. Login to the Web console of the DR site.

#. From the Async DR tab under Data Protection, select the Protection Domain that you want to Activate.

#. Right click on the protection domain - select **Activate**

    .. figure:: images/activate.png

#. Warning message appear: "Are you sure to activate protection domain?" - click **Yes**

#. Check recent tasks status and protection domain details - protection domain mode should be "active"

#. Look for your virtual machine and power it on

#. Log in to VM and check status and file content. Add current date-time to a file

Performing Failback
..................

#. The following steps would be completed by an instructor

#. Log on the primary site. Start all the hosts of the primary site. Controller VMs get started and the cluster configuration is established again.
        
#. All the Protection Domains that were active before the disaster occurred gets recreated in an active state. However, you cannot replicate the Protection Domains to the remote site since the Protection Domains are still active at the remote site.

#. Log on to one of the Controller VMs at the primary site and enter the hidden nCLI command mode by using the following nCLI command.
    nutanix@cvm:~$ ncli -h true

#. Deactivate and destroy the VMs by using the following hidden nCLI command. Run this command only when the Protection Domain is active on the secondary site because this command deletes the VMs from the primary site.
    ncli> pd deactivate-and-destroy-vms name=protection_domain_name

#. Caution: Do not use this command for any other workflow. Otherwise, it will delete the VMs and data loss will occur.

#. VMs get deleted from the primary site and the Protection Domain is no longer active on the primary site. Remove the orphaned VMs from the inventory of the primary site.

#. Perform the following steps:

#. Login to the Web console of the DR site (the site where the protection domain is currently active).

#. From the Async DR tab under Data Protection, select the Protection Domain that you want to failback.

#. Click **Migrate**. The Migrate Protection Domain dialog box appears. 

#. Select the site where you want to migrate the Protection Domain. The VMs that are part of the Protection Domain, but cannot be recovered on the remote site are also displayed. 

#. When the field entries are correct, click the Save button.

#. Once migration will be completed, login to primary site

#. In **Prism Element > Data Protection > Table**, select your protection domain

#. Check protection domain details - protection domain mode should be "active" 

    .. figure:: images/active.png

#. Power on your VM at the primary site, log in and check your text file. Add one more line with the current date and time.