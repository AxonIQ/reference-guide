# Backups

The core strategy employed by AxonServer to keep data available, is to replicate it over various cluster nodes. These nodes should be in availability zones that are isolated from each other in relevant disaster scenarios. With Axon Server Enterprise 4.3, the introduction of specific [Backup roles](backup-and-messaging-only-nodes.md) makes it easier to set-up and operate AxonServer without ever making explicit backups to off-line media. Nevertheless, there are also environments where such backups are a strict requirement and for that reason, AxonServer does support it.‌

There are three types of items that need to be backed up

* [Control Database](backups.md#control-database) - Axon SE/EE
* [Event Stream Segments](backups.md#event-stream-segments) - Axon SE/EE 
* [Log Entry Segments](backups.md#log-entry-segments) - Axon EE only

To support the creation of consistent backups, Axon Server provides a REST API. This API provides three controllers to perform backup operations

* _Backup Info Rest Controller_ - End point for Axon SE/EE for event stream segment backup
* _Backup Control DB Rest Controller_ - End point for Axon SE for control database backup
* _Cluster Backup Info Rest Controller_ -  End points for Axon EE for control database and log entry segment backup

The API documentation is accessible at http:\[server\]:\[port\]/swagger-ui.html.

## _Control Database_

The control database is a relational H2 database and contains important configuration information for your Axon Server SE/EE deployment. Although it's stored in a single file, this file cannot be simply copied for backup as it may not be in a safe state.

For Axon Server SE, a call to the POST endpoint `http://[server]/createControlDbBackup` forces the creation of a proper backup file.

For Axon Server EE, a call to the POST endpoint `http://[server]/v1/backup/createControlDbBackup`forces the creation of a proper backup file. The _\[server\]_ could be any node within the cluster which serves the _\_admin_ context.

In both cases, it returns the full path to that file \(.zip\), which can then be used to move that file to another storage medium.‌

## _Event Stream Segments_

The event stream segments are either closed and immutable, or still open for new events. For the closed segments, it is feasible to only backup the ones that haven't been backed-up yet, since the once that have been are guaranteed not to change.

For both Axon Server SE/EE, a call to the GET endpoint `http://[server]/v1/backup/filenames` with event type \(either `EVENT` or `SNAPSHOT`\), the context name and optionally the last segment that has already been backed up will return a list of file names belonging to segments that haven't been backed up yet, but which are now safe to backup by simply copying them.‌

For Axon SE, the _\[server\]_ is the single Axon Server SE node while in the case of Axon EE, the _\[server\]_ could be any node that is a PRIMARY member node for the context that needs to be backed up.

In addition, you may choose to backup the current segment file that is being written to. These are files with name larger than the last name returned in the filenames from the backup endpoint. It is important to overwrite this file with subsequent backups, because no guarantees can be given about the completeness of this file. This means the filename of this file should not be used to construct the "lastSegmentBackedUp" in subsequent requests to the backup endpoint.‌

## _Log Entry Segments \(only for Axon Server EE\)_

Unlike the event stream segments, the log entry segments backup should not be done incrementally. All the files are replaced by the next backup. The log entry segments backup is supported by the GET endpoint `http:[server]/v1/backup/log/filenames`. It takes the context name and returns a list of file names that completely replace the previous backup for that context.‌ The _\[server\]_ could be any node that is a PRIMARY member node for the context that needs to be backed up.

Even if the recent file has incomplete data, a node will be able to recover a consistent state from such a file and will initialize itself at the position immediately after the last complete write. The replication process \(if present\) will ensure subsequent entries are automatically synchronized.‌

Because the control database contains a pointer to the last log entry that is known to be stored safely on the cluster \(the commit index\), the proper order of doing this is to first create the control database backup and then backing up the log entry segments and the event stream segments.

This will ensure that the log entry segments may have entries beyond the commit index \(which is ok\) but there are not missing entries before the commit index \(which would be bad\). The log entries segments must be backed up within _**30 minutes**_ after the backup of the controlDB, to prevent the log compaction procedure causes data inconsistencies.

