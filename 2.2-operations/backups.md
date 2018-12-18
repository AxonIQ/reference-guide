# 2.2.1 Backup

The core strategy employed by AxonServer to keep data available, is to replicate it over various
cluster nodes. These nodes should be in availability zones that are isolated from each other in relevant disaster
scenarios. In such a set-up, it is feasible to operate AxonServer without ever making explicit backups
to off-line media. Nevertheless, there are also environments where such backups are a strict requirement
and for that reason, AxonServer does support it.

There are two types of items to be backed up: the control database, and the event stream segments. It is possible
to recover the data in an event store without the control database, but that might make recovery slower and therefore
we recommend to back it up.

Within the endpoints documented in /swagger-ui.html, there are two endpoints of the Backup Info Rest Controller.
They support in creating a consistent backup.

The control database is a relational H2 database. Although it's stored in a single file, this file cannot be simply
copied for backup as it may not be in a safe state. A call to the POST endpoint /v1/backup/createControlDbBackup
forces the creation of a proper backup file. It returns the full path to that file, which can then be used to
move that file to another storage medium.

The segment are either closed and immutable, or still open for new events. For the closed segments, it is feasible to
only backup the ones that haven't been backed-up yet, since the once that have been are guaranteed not to change. This
logic is supported by the GET endpoint /v1/backup/filenames. It takes an event type (either Event or Snapshot) and
optionally the last segment that has already been backed up. It will return a list of file names belonging to segments
that haven't been backed up yet, but which are now safe to backup by simply copying them.

In addition, you may choose to backup the current segment file that is being written to. These are files with name
larger than the last name returned in the filenames from the backup endpoint. It is important to overwrite this file
with subsequent backups, because no guarantees can be given about the completeness of this file. This means the filename
of this file should not be used to construct the "lastSegmentBackedUp" in subsequent requests to the
backup endpoint.

Even if the recent file has incomplete data, a node will be able to recover a consistent state from such a file and
will initialize itself at the position immediately after the last complete write. The replication process (if present)
will ensure subsequent entries are automatically synchronized.

Because the control database contains a pointer to the last event that is known to be stored safely on the cluster
(the _safepoint_), the proper order of doing this is to first create the control database backup and then backing
up the segments. This will ensure that the segments may have events beyond the safepoint (which is ok) but are
not missing events before the safepoint (which would be bad).
