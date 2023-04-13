# Operation to role mapping

The following table gives an overview of the roles in Axon Server and the operations that are granted to that role:

| Role                      | Operations                       |
|:--------------------------|:---------------------------------|
| Admin                     | DELETE_USER                      | 
|                           | DOWNLOAD_DIAGNOSE                |
|                           | DOWNLOAD_TEMPLATE                |
|                           | GET_APP_DETAILS                  |
|                           | GET_EVENT_PROCESSOR_STRATEGY     |
|                           | GET_EVENT_PROCESSORS             |
|                           | GET_EVENT_PROCESSORS_STRATEGIES  |
|                           | GET_PLUGIN_CONFIGURATION         |
|                           | INIT_CLUSTER                     |
|                           | LIST_APPS                        |
|                           | LIST_BACKUP0_FILENAMES           |
|                           | LIST_BACKUP_LOGFILES             |
|                           | LIST_COMMANDS                    |
|                           | LIST_CONTEXTS                    |
|                           | LIST_NODES                       |
|                           | LIST_PLUGINS                     |
|                           | LIST_QUERIES                     |
|                           | LIST_REPLICATION_GROUPS          |
|                           | LIST_TASKS                       |
|                           | LIST_USERS                       |
|                           | MERGE_USER                       |
|                           | RAFT_CLEAN_LOG                   |
|                           | RAFT_GET_STATUS                  |
|                           | RAFT_LIST_APPLICATIONS           |
|                           | RAFT_LIST_CONTEXT_MEMBERS        |
|                           | RAFT_LIST_CONTEXTS               |
|                           | RAFT_START_CONTEXT               |
|                           | RAFT_STEPDOWN                    |
|                           | RAFT_STOP_CONTEXT                |
|                           | REMOVE_NODE_FROM_CLUSTER         |
|                           | RENEW_APP_TOKEN                  |
|                           | UNREGISTER_PLUGIN                |
|                           | UPDATE_CONTEXT_PROPERTIES        |
|                           | UPLOAD_LICENSE                   |
| CONTEXT_ADMIN             | AUTO_REBALANCE_PROCESSOR         |
|                           | 0GET_EVENT_PROCESSOR_STRATEGY    |
|                           | 0GET_EVENT_PROCESSORS            |
|                           | 0GET_EVENT_PROCESSORS_STRATEGIES |
|                           | LIST_BACKUP_FILENAMES            |
|                           | LIST_BACKUP_LOGFILES             |
|                           | LOCAL_GET_LAST_EVENT             |
|                           | LOCAL_GET_LAST_SNAPSHOT          | 
|                           | MERGE_EVENT_PROCESSOR_SEGMENTS   | 
|                           | MOVE_EVENT_PROCESSOR_SEGMENT     | 
|                           | PAUSE_EVENT_PROCESSOR            | 
|                           | REBALANCE_PROCESSOR              |
|                           | 0RECONNECT_0CLIENT               |
|                           | SET_EVENT_PROCESSOR_STRATEGY     |
|                           | 0SPLIT_EVEN0T_PROCESSOR_SEGMENTS |
|                           | START_EVENT_PROCESSOR            |
| DISPATCH_COMMANDS         | DISPATCH_COMMAND                 |
| DISPATCH_QUERY            | DISPATCH_QUERY                   | 	
|                           | DISPATCH_SUBSCRIPTION_QUERY      |
| MONITOR                   | GET_COMMANDS_COUNT	              | 
|                           | GET_COMMANDS_QUEUE	              | 
| PUBLISH_EVENTS            | APPEND_EVENT                     | 	
|                           | APPEND_SNAPSHOT                  | 	
|                           | CANCEL_SCHEDULED_EVENT           | 
|                           | RESCHEDULE_EVENT                 | 
|                           | SCHEDULE_EVENT	                  | 
| READ                      | DISPATCH_QUERY                   |	
|                           | DISPATCH_SUBSCRIPTION_QUERY      |
|                           | GET_FIRST_TOKEN                  |
|                           | GET_LAST_TOKEN                   |
|                           | GET_TOKEN_AT                     |
|                           | HANDLE_QUERIES                   |
|                           | LIST_EVENTS                      |
|                           | LIST_SNAPSHOTS                   |
|                           | SEARCH_EVENTS                    | 
| READ_EVENTS               | GET_FIRST_TOKEN                  | 	
|                           | GET_LAST_TOKEN                   | 
|                           | GET_TOKEN_AT                     | 
|                           | LIST_EVENTS	                     | 
|                           | LIST_SNAPSHOTS	                  | 
|                           | READ_HIGHEST_SEQNR	              | 
|                           | SEARCH_EVENTS                    | 
| SUBSCRIBE_COMMAND_HANDLER | HANDLE_COMMANDS                  | 
| SUBSCRIBE_QUERY_HANDLER   | HANDLE_QUERIES                   |
| USE_CONTEXT               | APPEND_EVENT                     |
|                           | APPEND_SNAPSHOT	                 | 
|                           | AUTO_REBALANCE_PROCESSOR	        | 
|                           | CANCEL_SCHEDULED_EVENT	          | 
|                           | DISPATCH_COMMAND	                | 
|                           | DISPATCH_QUERY	                  | 
|                           | DISPATCH_SUBSCRIPTION_QUERY	     | 
|                           | GET_COMMANDS_COUNT	              | 
|                           | GET_COMMANDS_QUEUE	              | 
|                           | GET_EVENT_PROCESSOR_STRATEGY     | 
|                           | GET_EVENT_PROCESSORS	            |
|                           | GET_EVENT_PROCESSORS_STRATEGIES  | 
|                           | GET_FIRST_TOKEN                  |
|                           | GET_LAST_TOKEN                   |
|                           | GET_TOKEN_AT	                    |
|                           | HANDLE_COMMANDS	                 |
|                           | HANDLE_QUERIES	                  |
|                           | LIST_BACKUP_FILENAMES	           |
|                           | LIST_BACKUP_LOGFILES	            |
|                           | LIST_EVENTS	                     | 
|                           | LIST_QUERIES	                    | 
|                           | LIST_SNAPSHOTS	                  |
|                           | LOCAL_GET_LAST_EVENT	            | 
|                           | LOCAL_GET_LAST_SNAPSHOT	         | 
|                           | MERGE_EVENT_PROCESSOR_SEGMENTS   |
|                           | MOVE_EVENT_PROCESSOR_SEGMENT     |
|                           | PAUSE_EVENT_PROCESSOR	           |
|                           | READ_HIGHEST_SEQNR	              |
|                           | REBALANCE_PROCESSOR              | 
|                           | RECONNECT_CLIENT	                |
|                           | RESCHEDULE_EVENT	                |
|                           | SCHEDULE_EVENT	                  |
|                           | SEARCH_EVENTS	                   |
|                           | SET_EVENT_PROCESSOR_STRATEGY	    |
|                           | SPLIT_EVENT_PROCESSOR_SEGMENTS	  |
|                           | START_EVENT_PROCESSOR	           |
| VIEW_CONFIGURATION        | LIST_APPS	                       |
|                           | LIST_CONTEXTS	                   | 
|                           | LIST_NODES	                      |
|                           | LIST_PLUGINS	                    | 
|                           | LIST_REPLICATION_GROUPS	         |
|                           | LIST_USERS	                      | 
| WRITE                     | APPEND_EVENT	                    |
|                           | APPEND_SNAPSHOT	                 |
|                           | CANCEL_SCHEDULED_EVENT	          | 
|                           | DISPATCH_COMMAND	                |
|                           | HANDLE_COMMANDS	                 |
|                           | RESCHEDULE_EVENT	                |
|                           | SCHEDULE_EVENT	                  |

The following table gives an overview of the operations in Axon Server and the roles that can execute the operation:

| Operation	                          | Role                      |
|:------------------------------------|:--------------------------|
| ACTIVATE_PLUGIN	                    | ADMIN                     |
| ADD_NODE_TO_CLUSTER                 | 	ADMIN                    |
| ADD_NODE_TO_CONTEXT	                | ADMIN                     |
| ADD_NODE_TO_REPLICATION_GROUP	      | ADMIN                     | 
| ADD_PLUGIN	                         | ADMIN                     |
| APPEND_EVENT	                       | PUBLISH_EVENTS            |
|                                     | 	USE_CONTEXT              |
| 	                                   | WRITE                     |
| APPEND_SNAPSHOT	                    | PUBLISH_EVENTS            |
|                                     | 	USE_CONTEXT              |
|                                     | WRITE                     |
| AUTO_REBALANCE_PROCESSOR	           | CONTEXT_ADMIN             | 
|                                     | 	USE_CONTEXT              |
| CANCEL_SCHEDULED_EVENT	             | PUBLISH_EVENTS            |
|                                     | 	USE_CONTEXT              |
|                                     | 	WRITE                    |
| CONFIGURE_PLUGIN                    | 	ADMIN                    |
| CREATE_APP                          | 	ADMIN                    |
| CREATE_CONTEXT	                     | ADMIN                     |
| CREATE_CONTROLDB_BACKUP             | 	ADMIN                    |
| CREATE_REPLICATION_GROUP            | 	ADMIN                    |
| DELETE_APP	                         | ADMIN                     |
| DELETE_CONTEXT                      | 	ADMIN                    |
| DELETE_NODE_FROM_CONTEXT            | 	ADMIN                    |
| DELETE_NODE_FROM_REPLICATION_GROUP	 | ADMIN                     |
| DELETE_PLUGIN	                      | ADMIN                     |
| DELETE_REPLICATION_GROUP	           | ADMIN                     | 
| DELETE_TASK                         | 	ADMIN                    |
| DELETE_USER                         | 	ADMIN                    |
| DISPATCH_COMMAND                    | 	DISPATCH_COMMANDS        |
|                                     | 	USE_CONTEXT              |
| 	                                   | WRITE                     |
| DISPATCH_QUERY	                     | DISPATCH_QUERY            |
| 	                                   | READ                      |
| 	                                   | USE_CONTEXT               |
| DISPATCH_SUBSCRIPTION_QUERY	        | DISPATCH_QUERY            |
|                                     | 	READ                     |
|                                     | 	USE_CONTEXT              |
| 0DOWNLOAD_DIAGNOSE	                 | ADMIN                     | 
| 0DOWNLOAD_TEMPLATE	                 | ADMIN                     | 
| GET_APP_DETAILS	                    | ADMIN                     |
| GET_COMMANDS_COUNT                  | 	MONITOR                  |
|                                     | 	USE_CONTEXT              |
| GET_COMMANDS_QUEUE                  | 	MONITOR                  |
|                                     | 	USE_CONTEXT              |
| GET_EVENT_PROCESSOR_STRATEGY        | 	ADMIN                    | 
|                                     | 	CONTEXT_ADMIN            | 
|                                     | 	USE_CONTEXT              | 
| GET_EVENT_PROCESSORS	               | ADMIN                     |
| 	                                   | CONTEXT_ADMIN             |
| 	                                   | USE_CONTEXT               |
| GET_EVENT_PROCESSORS_STRATEGIES	    | ADMIN                     | 
| 	                                   | CONTEXT_ADMIN             | 
| 	                                   | USE_CONTEXT               | 
| GET_FIRST_TOKEN	                    | READ                      |
| 	                                   | READ_EVENTS               |
| 	                                   | USE_CONTEXT               |
| GET_LAST_TOKEN	                     | READ                      |
| 	                                   | READ_EVENTS               | 
| 	                                   | USE_CONTEXT               | 
| GET_PLUGIN_CONFIGURATION	           | ADMIN                     | 
| GET_TOKEN_AT	                       | READ                      |
|                                     | 	READ_EVENTS              |
| 	                                   | USE_CONTEXT               |
| HANDLE_COMMANDS	                    | SUBSCRIBE_COMMAND_HANDLER | 
| 	                                   | USE_CONTEXT               | 
| 	                                   | WRITE                     | 
| HANDLE_QUERIES	                     | READ                      | 
| 	                                   | SUBSCRIBE_QUERY_HANDLER   | 
| 	                                   | USE_CONTEXT               | 
| INIT_CLUSTER	                       | ADMIN                     | 
| LIST_APPS	                          | ADMIN                     |
| 	                                   | VIEW_CONFIGURATION        |
| LIST_BACKUP_FILENAMES	              | ADMIN                     | 
| 	                                   | CONTEXT_ADMIN             | 
| 	                                   | USE_CONTEXT               | 
| LIST_BACKUP_LOGFILES	               | ADMIN                     | 
| 	                                   | CONTEXT_ADMIN             | 
| 	                                   | USE_CONTEXT               | 
| LIST_COMMANDS	                      | ADMIN                     | 
| LIST_CONTEXTS	                      | ADMIN                     | 
| 	                                   | VIEW_CONFIGURATION        | 
| LIST_EVENTS	                        | READ                      | 
| 	                                   | READ_EVENTS               | 
| 	                                   | USE_CONTEXT               | 
| LIST_NODES	                         | ADMIN                     | 
| 	                                   | VIEW_CONFIGURATION        | 
| LIST_PLUGINS	                       | ADMIN                     | 
| 	                                   | VIEW_CONFIGURATION        | 
| LIST_QUERIES	                       | ADMIN                     | 
| 	                                   | USE_CONTEXT               | 
| LIST_REPLICATION_GROUPS	            | ADMIN                     | 
| 	                                   | VIEW_CONFIGURATION        | 
| LIST_SNAPSHOTS	                     | READ                      | 
| 	                                   | READ_EVENTS               | 
| 	                                   | USE_CONTEXT               | 
| LIST_TASKS	                         | ADMIN                     |
| LIST_USERS	                         | ADMIN                     |
| 	                                   | VIEW_CONFIGURATION        |
| LOCAL_GET_LAST_EVENT	               | CONTEXT_ADMIN             | 
| 	                                   | USE_CONTEXT               | 
| LOCAL_GET_LAST_SNAPSHOT	            | CONTEXT_ADMIN             | 
| 	                                   | USE_CONTEXT               | 
| MERGE_EVENT_PROCESSOR_SEGMENTS	     | CONTEXT_ADMIN             |
| 	                                   | USE_CONTEXT               | 
| MERGE_USER	                         | ADMIN                     | 
| MOVE_EVENT_PROCESSOR_SEGMENT	       | CONTEXT_ADMINv            |
| 	                                   | USE_CONTEXT               | 
| PAUSE_EVENT_PROCESSOR	              | CONTEXT_ADMIN             |
| 	                                   | USE_CONTEXT               |
| RAFT_CLEAN_LOG	                     | ADMIN                     |
| RAFT_GET_STATUS	                    | ADMIN                     |
| RAFT_LIST_APPLICATIONS	             | ADMIN                     | 
| RAFT_LIST_CONTEXT_MEMBERS	          | ADMIN                     |
| RAFT_LIST_CONTEXTS	                 | ADMIN                     | 
| RAFT_START_CONTEXT	                 | ADMIN                     | 
| RAFT_STEPDOWN	                      | ADMIN                     |
| RAFT_STOP_CONTEXT	                  | ADMIN                     |
| READ_HIGHEST_SEQNR	                 | READ_EVENTS               |
| 	                                   | USE_CONTEXT               |
| REBALANCE_PROCESSOR	                | CONTEXT_ADMIN             |
| 	                                   | USE_CONTEXT               |
| RECONNECT_CLIENT	                   | CONTEXT_ADMIN             |
| 	                                   | USE_CONTEXT               |
| REMOVE_NODE_FROM_CLUSTER	           | ADMIN                     |
| RENEW_APP_TOKEN	                    | ADMIN                     |
| RESCHEDULE_EVENT	                   | PUBLISH_EVENTS            |
| 	                                   | USE_CONTEXT               |
| 	                                   | WRITE                     |
| SCHEDULE_EVENT	                     | PUBLISH_EVENTS            |
| 	                                   | USE_CONTEXT               | 
| 	                                   | WRITE                     | 
| SEARCH_EVENTS	                      | READ                      |
| 	                                   | READ_EVENTS               |
| 	                                   | USE_CONTEXT               |
| SET_EVENT_PROCESSOR_STRATEGY	       | CONTEXT_ADMIN             |
| 	                                   | USE_CONTEXT               | 
| SPLIT_EVENT_PROCESSOR_SEGMENTS      | CONTEXT_ADMIN             | 
|                                     | USE_CONTEXT               | 
| START_EVENT_PROCESSOR	              | CONTEXT_ADMIN             | 
| 	                                   | USE_CONTEXT               | 
| UNREGISTER_PLUGIN	                  | ADMIN                     | 
| UPDATE_CONTEXT_PROPERTIES	          | ADMIN                     | 
| UPLOAD_LICENSE	                     | ADMIN                     | 