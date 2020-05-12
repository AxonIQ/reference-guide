# F. Axon Server APIs

Axon Server provides a comprehensive REST based API to interact with and perform operations against both standard and enterprise editions. 

We have also provided a Postman collection to run these against your deployment of Axon Server. You would need to change the {{base\_url}} which would point to a Axon Server SE node or in the case of an Axon Server EE cluster, the admin node of the cluster. For token based authorization, please add the following header -&gt; _AxonIQ-Access-Token_

| Object | API Purpose |  |
| :--- | :--- | :--- |
| _applications_ | Perform operations against registered Axon Framework applications | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _backup_ | Take backups of the control database | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _cluster_ | Cluster Maintenance for an Axon Server EE deployment | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _commands_ | Details of executed commands on an Axon Server | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _components_ | Component Maintenance for an Axon Server deployment  | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _context_ | Context Maintenance for an Axon Server EE deployment | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _events_ | Details of executed events on an Axon Server | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _local_ |  | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _processors_ | Event Processor Maintenance for an Axon Server deployment | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _public_ | Operational metrics for an Axon Server | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _queries_ | Details of executed queries on an Axon Server | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _roles_ | Details of the roles setup on an Axon Server  | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _snapshots_ | Snapshot Details for an Axon Server deployment | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |
| _users_ | Details of the users setup on an Axon Server | [![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/67c1bdfd23e5498a9cb8) |

