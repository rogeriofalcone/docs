Backup
-------

Provides interface for database backup and restore management, scheduling and reporting. Each backup will be assigned with a backup ID and ClusterControl creates a directory under *Storage Directory* to store the backups based on this ID. On top of the page, you can see 3 function tabs followed by the created backup list underneath it.

Create Backup
`````````````

Creates or schedules a PostgreSQL backup. 

Create Backup
.............

You can choose to create a full backup using :term:`pg_dumpall` or :term:`pg_basebackup`. Backups can be stored on the database host that is performing the backup, or the files can be streamed to the ClusterControl host. The backup created by this feature will be a full backup.

.. Note:: The backup created by ClusterControl for PostgreSQL is always a full compressed backup.

* **Backup Method**
	- pgdump - Full compressed backup using ``pg_dumpall``. See `pg_dumpall`_ section.
	- pg_basebackup - Full compressed backup using ``pg_basebackup``. See `pg_basebackup`_ section.

* **Backup Host**
	- The target database host.
	
* **Storage Location**
	- Store on Node - Stores the backup inside corresponding database node.
	- Store on Controller - Stores the backup inside ClusterControl node. This requires :term:`socat` or :term:`netcat` on source and destination host. By default, ClusterControl uses port 9999 to stream the backup created on the database node to ClusterControl node.

* **Storage Directory**
	- You can opt to use another backup directory as you wish. If you leave this field blank, ClusterControl will use the default backup directory specified in the *Settings > Default backup directory*.
	
* **Netcat Port**
	- Specify the port number that will be used by ClusterControl to stream backup created on the database node. This port must be opened on both source and destination hosts. Only available if you choose *Store on Controller* in *Storage Location*.
	
* **Use Compression**
	- Yes - Tells the chosen backup method to compress all output data, including the transaction log file and meta data files.
	- No - Do not use compression for the backup.

* **Compression Level**
	- Specify the compression level for the backup. This is according to :term:`gzip` compression level. 1 is the fastest compression (least compression) and 9 indicates the slowest compression (best compression).

* **Upload Backup to cloud**
	- Automatically upload the finished backup to AWS S3 or Google Cloud Storage. This backup can then be downloaded and restored from the cloud. You have to configure `Cloud Credentials <../index.html#cloud-providers>`_ beforehand.

* **Enable Encryption**
	- Encrypts the generated backup. Backup is encrypted at rest using AES-256 CBC algorithm, where the encryption key will be created automatically. If you choose to store the backup on the controller node, the backup files are transferred in encrypted format through :term:`socat` or :term:`netcat`.

* **Retention**
	- How long ClusterControl should keep this backup once successfully created. You can set a custom period in days or keep it forever. Otherwise, ClusterControl will use the default retention period.


Schedule Backup
................

Creates backup schedules of the database.

* **Schedule**
	- Simple - Default scheduling option. This translates to the same output as the Advanced editor.
	- Advanced - Opens a cron-like editor. Formatting is similar to the standard :term:`cron`.

.. Note:: The backup time is in UTC time zone of the ClusterControl node.

* **Backup Method**
	- pgdump - Full compressed backup using ``pg_dumpall``. See `pg_dumpall`_ section.
	- pg_basebackup - Full compressed backup using ``pg_basebackup``. See `pg_basebackup`_ section.

* **Backup Host**
	- The target database host.

* **Storage Location**
	- Store on Node - Stores the backup inside corresponding database node.
	- Store on Controller - Stores the backup inside ClusterControl node. This requires :term:`socat` or :term:`netcat` on source and destination host. By default, ClusterControl uses port 9999 to stream the backup created on the database node to ClusterControl node.

* **Storage Directory**
	- You can opt to use another backup directory as you wish. If you leave this field blank, ClusterControl will use the default backup directory specified in the *Settings > Default backup directory*.

* **Netcat Port**
	- Specify the port number that will be used by ClusterControl to stream backup created on the database node. This port must be opened on both source and destination hosts. Only available if you choose *Store on Controller* in *Backup Location*.

* **Use Compression**
	- Yes - Tells the chosen backup method to compress all output data, including the transaction log file and meta data files.
	- No - Do not use compression for the backup.

* **Compression Level**
	- Specify the compression level for the backup. This is according to :term:`gzip` compression level. 1 is the fastest compression (least compression) and 9 indicates the slowest compression (best compression).

* **Failover backup if node is down**
	- Yes - Backup will be run on any available node (or selected node based on the *Backup Failover Host*) if the target database node is down. If failover is enabled and the selected node is not online, the backup job elects an online node to create the backup. This ensures that a backup will be created even if the selected node is not available. If the scheduled backup is an incremental backup and a full backup does not exist on the new elected node, then a full backup will be created.
	- No - Backup will not run if the target database node is down.
	
* **Failover Host**
	- List of database host to failover in case the target node is down during the scheduled backup.

* **Enable Encryption**
	- Encrypts the generated backup. Backup is encrypted at rest using AES-256 CBC algorithm, where the encryption key will be created automatically. If you choose to store the backup on the controller node, the backup files are transferred in encrypted format through :term:`socat` or :term:`netcat`.

* **Retention**
	- How long ClusterControl should keep this backup once successfully created. You can set a custom period in days or keep it forever. Otherwise, ClusterControl will use the default retention period.
  
Scheduled Backups
`````````````````

List of scheduled backups. You can enable and disable the schedule by toggling it accordingly. The created schedule can be edited and deleted.

Backup Method
`````````````

This section explains backup method used by ClusterControl.

.. Note:: Backup process performed by ClusterControl is running as a background thread (RUNNING3) which doesn't block any other non-backup jobs in queue. If the backup job takes hours to complete, other non-backup jobs can still run simultaneously via the main thread (RUNNING). You can see the job progress at *ClusterControl > Logs > Jobs*.

pg_dump
.........

ClusterControl performs :term:`pg_dumpall` against all databases together with ``--clean`` option, which include SQL commands to clean (drop) databases before recreating them. DROP commands for roles and tablespaces are added as well. The output will be in ``.sql.gz`` extention and file name contains the timestamp of the backup.

pg_basebackup
..............

pg_basebackup is used to take base backups of a running PostgreSQL database cluster. These are taken without affecting other clients to the database, and can be used both for point-in-time recovery and as the starting point for a log shipping or streaming replication standby servers. It makes a binary copy of the database cluster files, while making sure the system is put in and out of backup mode automatically. Backups are always taken of the entire database cluster; it is not possible to back up individual databases or database objects.

ClusterControl connects to the replication stream using the replication user (default is ``cmon_replication``) with ``--wal-method=fetch`` option when creating the backup. The output will be ``base.tar.gz`` inside the backup directory.

Backup List
````````````

Provides a list of finished backup jobs. The status can be:

========= ===========
Status    Description
========= ===========
Completed Backup was successfully created and stored in the chosen node.
Running   Backup process is running.
Failed    Backup was failed.
========= ===========

* **Restore**
	- See `Restore Backup`_.

* **Log**
	- Shows the output once ClusterControl executes the backup job.

* **Delete**
	- Removes the backup set.

* **Upload**
	- Manually upload the created backup to cloud storage. This will open "Upload Backup" wizard.

Restore Backup
``````````````

Restores ``pg_dump`` or ``pg_basebackup`` backup file created by ClusterControl and listed in the `Backup List`_. 

Restore on node
.................

You can restore up to a certain incremental backup by clicking on the *Restore* button for the respective backup ID. The following steps will be performed:

For pgdump (online restore):

1. Copy backup files to the target server.
2. Checking disk space on the target server.
3. Restore the backup.
4. Follow the instruction in the *ClusterControl > Activity > Jobs* on how to rebuild the slaves.

For pg_basebackup (offline restore):

1. Stop the target node.
2. Backup the current PostgreSQL data directory.
3. Copy backup files to the target server.
4. Checking disk space on the target server.
5. Prepare and restore the backup.
6. Start the target node.
5. Follow the instruction in the *ClusterControl > Activity > Jobs* on how to rebuild the slaves.

* **Restore backup on**
	- The backup will be restored to the selected server.
	
* **Tmp Dir**
	- Temporary storage for ClusterControl to prepare the big. It must be as big as the expected PostgreSQL data directory.
	
.. Attention:: The data directory must have enough space to accommodate the restored backup.

Settings
````````

Manages the backup settings.

* **Default backup directory**
	- Default path for the backup directory. ClusterControl will create the backup directory on the destination host if doesn't exist.

* **Backup retention period**
	- The number of days ClusterControl keeps the existing backups. Backups older than the value defined here will be deleted. You can also customize the retention period per backup (default, custom or keep forever) under *Backup Retention* when creating or scheduling the backup.

* **Backup cloud retention period**
	- The number of days ClusterControl keeps the uploaded backups in the cloud. Backups older than the value defined here will be deleted.