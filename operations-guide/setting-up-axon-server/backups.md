# Backups

The core strategy employed by AxonServer to keep data available, is to replicate it over various
cluster nodes. These nodes should be in availability zones that are isolated from each other in relevant disaster
scenarios. In such a set-up, it is feasible to operate AxonServer without ever making explicit backups
to off-line media. Nevertheless, there are also environments where such backups are a strict requirement
and for that reason, AxonServer does support it.

There are three types of items to be backed up: the control database, the event stream segments and the log entry
segments. 

Within the endpoints documented in `/swagger-ui.html`, there are two endpoints of the Backup Info Rest Controller 
and one endpoint of the Backup Log Files Rest Controller, available only in the AxonServer Enterprise version.
They support in creating a consistent backup.

The control database is a relational H2 database. Although it's stored in a single file, this file cannot be simply
copied for backup as it may not be in a safe state. A call to the POST endpoint `/v1/backup/createControlDbBackup`
forces the creation of a proper backup file. It returns the full path to that file, which can then be used to
move that file to another storage medium.

The event stream segments are either closed and immutable, or still open for new events. For the closed segments, it is 
feasible to only backup the ones that haven't been backed-up yet, since the once that have been are guaranteed not to 
change. This logic is supported by the GET endpoint `/v1/backup/filenames`. It takes an event type (either Event or 
Snapshot), the context name and optionally the last segment that has already been backed up. It will return a list of 
file names belonging to segments that haven't been backed up yet, but which are now safe to backup by simply copying them.

In addition, you may choose to backup the current segment file that is being written to. These are files with name
larger than the last name returned in the filenames from the backup endpoint. It is important to overwrite this file
with subsequent backups, because no guarantees can be given about the completeness of this file. This means the filename
of this file should not be used to construct the "lastSegmentBackedUp" in subsequent requests to the
backup endpoint.

Unlike the event stream segments, the log entry segments backup should not be done incrementally. All the files are
replaced by next backup. The log entry segments backup is supported by the GET endpoint `/v1/backup/log/filenames`. 
It takes the context name and returns a list of file names that completely replace the previous backup for that context.

Even if the recent file has incomplete data, a node will be able to recover a consistent state from such a file and
will initialize itself at the position immediately after the last complete write. The replication process (if present)
will ensure subsequent entries are automatically synchronized.

Because the control database contains a pointer to the last log entry that is known to be stored safely on the cluster 
(the commit index), the proper order of doing this is to first create the control database backup and then backing up 
the log entry segments and the event stream segments. This will ensure that the log entry segments may have entries 
beyond the commit index (which is ok) but there are not missing entries before the commit index (which would be bad).
The log entries segments must be backed up within 30 minutes after the backup of the controlDB, to prevent the log 
compaction procedure causes data inconsistencies.
