# Standard to Enterprise Edition

There are multiple benefits of moving from an Axon Server SE based deployment to an Axon Server EE based deployment and Axon provides an easy mechanism to achieve this. The up-gradation process primarily involves the movement of the Event/Snapshot data from the an Axon Server SE node to the Axon Server EE cluster.

## Pre-Upgrade Process

* All Event/Snapshots Data \(\*.events / \*.snapshots\) in Axon Server SE will be under the folder _${axon\_se\_server\_home}/data/default_
* Provision the Axon Server EE cluster with the required number of nodes setup for the default context.
* Once this is completed and validated, shut down all the nodes within the cluster

## Upgrade process

* Copy and Replace the Event/Snapshots data from the single SE node to every node on the EE cluster. The files should be copied to the\_${axon\_ee\_server\_home}/data/default\_ location to each of the EE nodes
* Restart the Nodes

## Verification

* Logon to the Axon Server EE console and query the store to check if the migrated event data is present

## Notes

* The controldb file on Axon Server SE just contains information on the users allowed access to the server. Users in Axon Server SE are not assigned to any contexts or roles. It is recommended that this file is not copied over over and these users be recreated in Axon Server EE and assigned the proper roles/contexts.
* Any token setup for client access to the Axon Server SE must be setup on the Axon Server EE cluster too.

