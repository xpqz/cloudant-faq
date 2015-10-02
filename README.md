# cloudant-faq
Collection of questions and answers gathered in the line of duty

1. Does Cloudant have a mechanism that can trigger data pull from from the database using a callback-based observer pattern?
`No`
1. Can Cloudant mobile use an existing TLS transport?  
```No – we support platform-standard HTTPS libraries only```

1. Is Cloudant (both server and mobile) FIPS 140-2 compliant? We have our own FIPS compliant library, can we use that with Cloudant?  
```No/no - but it’s a high priority on our road map. Can’t bring your own library at this point```

1. In terms of data encryption for Cloudant mobile – what are the strengths and processing overhead?  
```AES/CBC 256bit key, 1.2x slowdown for normal document work. 3x slowdown for indexing large numbers of docs at once```

1. Are there APIs that can update a specific value for a key without reading/re-writing the whole document?  
`No`

1. If we change a document on Cloudant, is the payload for sync the whole document or the delta changes only?  
```Whole document```

1. Can we do filtered replication at the doc level, directly between Cloudant and server?  
```Yes, by writing custom JavaScript filter function for changes feed (running a filtered replication in CouchDB-terms)```

1. How are attachments handled for the sync mechanism? What happens when the document is updated but the attachment is not after the initial sync?  
```If you change the document but not the attachments, only the document JSON is replicated, not the attachments```

1. If the mobile user is offline and multiple updates happened locally to the document, when the sync resumes will it sync the final revision or all the revisions?  
```Syncs final document content only, with a small amount of revision history meta-data to maintain the tree structure```

1. How is sync performance impacted? What are the limiting factors?  
```Limiting factors: network latency and bandwidth. A large amount of tiny documents will make this worse due to the HTTP overhead```

1. How are attachments stored in Cloudant/SQLite?  
```On device, each attachment is stored as a blob on disk, not in SQLite. Attachments with identical content are only stored once on disk, and referenced. External developers do not have access to the raw data on disk.```

1. What is the max doc/attachment size limit in the mobile library?  
```No theoretical hard limit, but in practice each document is stored as a row in SQLite, so larger documents have a performance implication based on SQLite. For attachments, it depends on what kind of connection you’re using for sync - they’re always sent whole, and the transfer isn’t resumable.```

1. What is the plan to support other mobile platforms like Windows mobile, Firefox, BB10?  
```Although other mobile platforms are on our radar, we have had little in terms of customer demands for platforms outside iOS and Android, so as a consequence we currently have no concrete plans.```

1. How do we enable Cloudant Mobile to support closed OS phones? Do you think its portable to features phones running closed OS?  
```We’d imagine this to be difficult.```

1. What is an easy way to delete a local copy of the document without removing the “original” doc from the server?  
```There is no good solution to this right now. Internally we have a purge function that does this, but it’s not currently exposed - although if there is a need for this we would consider exposing it.```

1. Do you have version management for the document conflict resolution on the mobile side? `No`

1. Any support for websocket transport for iOS and Android? `No`

1. Do you offer Ipv6 support?  
```We effectively rely on the underlying HTTP/ip libraries to resolve this for us.```

1. Is there a C SDK for mobile? `No`

1. How does Cloudant support PouchDB?  
```PouchDB is tested, and works with Cloudant. IBM does not officially offer support for PouchDB, but we do strive to maintain compatibility and have committed patches to improve compatibility between Cloudant and PouchDB. We view PouchDB as a vital part of the larger CouchDB ecosystem.```

1. How is client redundancy achieved?  
```To maintain the ability of a client to sync data we typically use two or more geographically distributed clusters which sync data with each other, thereby allowing clients to talk to either cluster in case the other is unreachable. This can be supported by transparent redirection on the server side or a device can explicitly replicate with more than one cluster```
    
1. What authentication and authorization mechanisms are provided for mobile users to connect to the server?  
```At the moment we support HTTP-basic-auth over HTTPS. We’re currently working on other auth mechanisms.```

1. Does Cloudant support digest based authentication for sync?  `No`

1. Does Cloudant support cross data center replication and clusters?  
```We do support cross-datacenter replication. Typically we recommend that clients do not have clusters that span multiple data centres, but instead have multiple clusters that replicate between them. However, we do support clusters spanning multiple data centres using zones to ensure each data centre have at least one complete replica.```

1. Can we add more nodes to the cluster without any outage to the service? What is the mechanism?  
```Yes. It’s done by asking us to add nodes to the system - we will provision them and add them to the cluster. We can also handle rebalancing if required.```

1. We are trying to model an application service that listens for database notifications. For scalability and redundancy we will have multiple instances of the service. What capabilities does Cloudant offer in this regard?  
```Each instance of the service would need to listen to the _changes feed. Alternatively, a single instance could listen to the feed and post updates to a message queue system (e.g. RabbitMQ) where worker processes could pick up messages to process.```

1. Is CORS supported? Is webhooks supported for data changes?  
```CORS is supported. Web hooks are not.```

1. What are the deployment options of Cloudant and their differences?  
```Cloudant can be deployed as Multi-Tenant (MT), Dedicated, or Local. MT means that cluster resources are shared between different, unrelated customers which may have performance implications. A dedicated deployment means that a cluster is private to you, and sized to your requirements, but still fully managed by Cloudant, 24x7. The Local option means that you can run a Cloudant cluster from within the confines of your own data center, giving you full control, but also the management responsibility.```

1. How are the different options priced?  
```Cloudant MT is PAYGO based on usage. Dedicated is a flat monthly fee based on the number and spec of nodes. Local is a per-node annual license fee, with a 1-node license suitable for development included in MobileFirst Platform (MFP).```

1. What are the differentiators of Cloudant? What are the competitive advantages of Cloudant against the competitors? Who are they?  
```Cloudant is a schema-free, distributed, master-less replicating cloud database. Its strongest competitive advantages are its replication capabilities, its durability and its availability. In simplified terms, we can basically yank the power cable of a node in a cluster and the system would be unaffected. Once the node comes back, it would quietly catch up through replication. The NoSQL database space has a large number of different products, all making their particular trade-offs. Such databases (including Cloudant) tend to be more use-case specific than traditional RDBMS. We sometimes encounter MongoDB as a competitor in the field, even though it’s really designed for different goals. Mongo’s storage model is such that it can be seen as more fragile than Cloudant – when Cloudant acknowledges a write, you are guaranteed that the data is written to disk in multiple copies whereas Mongo can be tuned for speed at the expense of durability. Other competitors are Amazon’s DynamoDB, and Couchbase.```

1. How would Cloudant be provisioned with data (initial provisioning, regular batch and real time against a backend)?  
```Cloudant has a simple, easy to use HTTP API for getting data into and out of a cluster. Several IBM ETL tools exist that can be helpful when migrating large data volumes into Cloudant.```

1. How would Cloudant be integrated to the backend?  
```It depends on the use case and deployment model. Cloudant has a sophisticated ‘changes’ API end-point which provides incremental access to database updates. This can be used to implement various kinds of integrations by using it to incrementally update external systems (e.g., ElasticSearch, external SoR systems). A common pattern we’ve seen is that the master copy of data lives in an existing enterprise system of record database. Cloudant is used as a scalable method to project this data to many mobile devices, either read-only or read-write. In the read-only case, a database trigger or other method must be used to keep the data in Cloudant up to date using Cloudant’s HTTP API. If read-write is needed, more sophisticated methods need using as the enterprise system will usually not support incremental sync out-of-the-box. Using Cloudant as the SoR gives the most flexible and easiest-to-manage solution.```

1. How would Cloudant be managed (initial size and elastic extension, backup etc)?  
```It depends on the deployment model. Initial size is modeled on the application’s properties: number of users; read vs. write rate; data volumes etc. A typical 3-node dedicated cluster can support of the order of 1,000-1,500 HTTP requests per second. Elastic cluster expansion isn’t fully automatic; growing a cluster requires some operational intervention. For dedicated and MT deployment the management of the cluster is handled by Cloudant (patching, monitoring etc). Backups are less important in a replicating, master-less distributed system such as Cloudant as data is stored in triplicate by default. If further backup is required it is straightforward to commission a backup cluster (in a different data center, for example) to which data is replicated continuously. The latter option also provides for snapshot backups. More info on backups here: https://ibm.box.com/cloudant-backup-dr```

1. How is Cloudant be secured (data encryption on device, on Cloudant node etc)  
```Cloudant’s mobile libraries have the option to encrypt data at rest, and data in flight is always over a secure channel. On a dedicated Cloudant database node, data isn’t encrypted at rest by default, but work is underway to provide an option with full FIPS140-2 compliance on the server. Cloudant Local is of course secured in your own data centers; disk encryption can be used if needed. In addition, management of the cluster nodes is restricted to those with an access need and we obviously keep our systems fully patched.```

1. How would Cloudant resolve conflicts on multiple updates on different node within a cluster?  
```Cloudant has industry-leading conflict handling. Cloudant uses a data model called MVCC, multi-version concurrency control, meaning that each document can exist in several versions. To a certain degree, this is similar to a distributed master-less revision control system such as ‘git’. It means that concurrent modification is a fully supported state of the data, rather than an exception. Finally, no data is ever discarded unless explicitly superseded by business logic. As a consequence, conflict resolution can be deferred to the point where it makes most sense to the application’s business logic. In a distributed, replicated system the database cannot safely attempt to resolve a data conflict without sacrificing availability. Cloudant provides API to applications using it which support detecting and resolving conflicts.```

1. How would Cloudant synchronize with different DR site? What is the requirement on the network link?  
```Cloudant is fully set up to do inter-datacenter replication. The draw on the network link depends on many factors: data update frequency, data volume. Traffic goes over a HTTPS link, so all that’s required is a network connection between the two (or more) clusters involved in DR. More DR info can be found here: https://ibm.box.com/cloudant-backup-dr```

1. How does Cloudant geospatial query?  
```Geo is a huge topic. Cloudant supports geo-spatial querying and supports GeoJSON, GeoPackage and Web Feature Services. A good introductory overview can be found here: https://ibm.box.com/cloudant-geo-overview, and details of the geo-spatial API can be found here: https://cloudant.com/wp-content/uploads/Cloudant-Geospatial-API-documentation.pdf```








