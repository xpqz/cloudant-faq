# cloudant-faq
## Collection of questions and answers gathered in the line of duty

1. _Does Cloudant have a mechanism that can trigger data pull from from the database using a callback-based observer pattern?_  
    No.  

1. _Can Cloudant mobile use an existing TLS transport?_  
    No – we support platform-standard HTTPS libraries only.

1. _Is Cloudant (both server and mobile) FIPS 140-2 compliant? We have our own FIPS compliant library, can we use that with Cloudant?_  
    No/no - but it’s a high priority on our road map. Can’t bring your own library at this point.

1. _In terms of data encryption for Cloudant mobile – what are the strengths and processing overhead?_  
    AES/CBC 256bit key, 1.2x slowdown for normal document work. 3x slowdown for indexing large numbers of docs at once.

1. _Are there APIs that can update a specific value for a key without reading/re-writing the whole document?_  
    No.

1. _If we change a document on Cloudant, is the payload for sync the whole document or the delta changes only?_  
    Whole document.

1. _Can we do filtered replication at the doc level, directly between Cloudant and server?_  
    Yes, by writing custom JavaScript filter function for changes feed (running a filtered replication in CouchDB-terms).

1. _How are attachments handled for the sync mechanism? What happens when the document is updated but the attachment is not after the initial sync?_  
    If you change the document but not the attachments, only the document JSON is replicated, not the attachments.

1. _If the mobile user is offline and multiple updates happened locally to the document, when the sync resumes will it sync the final revision or all the revisions?_  
    Syncs final document content only, with a small amount of revision history meta-data to maintain the tree structure.

1. _How is sync performance impacted? What are the limiting factors?_  
    Limiting factors: network latency and bandwidth. A large amount of tiny documents will make this worse due to the HTTP overhead.

1. _How are attachments stored in Cloudant/SQLite?_  
    On device, each attachment is stored as a blob on disk, not in SQLite. Attachments with identical content are only stored once on disk, and referenced. External developers do not have access to the raw data on disk.

1. _What is the max doc/attachment size limit in the mobile library?_  
    No theoretical hard limit, but in practice each document is stored as a row in SQLite, so larger documents have a performance implication based on SQLite. For attachments, it depends on what kind of connection you’re using for sync - they’re always sent whole, and the transfer isn’t resumable.

1. _What is the plan to support other mobile platforms like Windows mobile, Firefox, BB10?_  
    Although other mobile platforms are on our radar, we have had little in terms of customer demands for platforms outside iOS and Android, so as a consequence we currently have no concrete plans.

1. _How do we enable Cloudant Mobile to support closed OS phones? Do you think its portable to feature phones running closed OS?_  
    We’d imagine this to be difficult.

1. _What is an easy way to delete a local copy of the document without removing the “original” doc from the server?_  
    There is no good solution to this right now. Internally we have a purge function that does this, but it’s not currently exposed - although if there is a need for this we would consider exposing it.

1. _Do you have version management for the document conflict resolution on the mobile side?_ 
    No.

1. _Any support for websocket transport for iOS and Android?_ 
    No.

1. _Do you offer Ipv6 support?_  
    We effectively rely on the underlying HTTP/ip libraries to resolve this for us.

1. _Is there a C SDK for mobile?_
    No.

1. _How does Cloudant support PouchDB?_  
    PouchDB is tested, and works with Cloudant. IBM does not officially offer support for PouchDB, but we do strive to maintain compatibility and have committed patches to improve compatibility between Cloudant and PouchDB. We view PouchDB as a vital part of the larger CouchDB ecosystem.

1. _How is client redundancy achieved?_  
    To maintain the ability of a client to sync data we typically use two or more geographically distributed clusters which sync data with each other, thereby allowing clients to talk to either cluster in case the other is unreachable. This can be supported by transparent redirection on the server side or a device can explicitly replicate with more than one cluster.
    
1. _What authentication and authorization mechanisms are provided for mobile users to connect to the server?_  
    At the moment we support HTTP-basic-auth and session cookies over HTTPS. We’re currently working on other auth mechanisms.

1. _Does Cloudant support digest based authentication for sync?_
    No

1. _Does Cloudant support cross data center replication and clusters?_  
    We do support cross-datacenter replication. Typically we recommend that clients do not have clusters that span multiple data centres, but instead have multiple clusters that replicate between them. However, we do support clusters spanning multiple data centres using zones to ensure each data centre have at least one complete replica.

1. _Can we add more nodes to the cluster without any outage to the service? What is the mechanism?_  
    Yes. It’s done by asking us to add nodes to the system – we will provision them and add them to the cluster. We can also handle rebalancing if required.

1. _We are trying to model an application service that listens for database notifications. For scalability and redundancy we will have multiple instances of the service. What capabilities does Cloudant offer in this regard?_  
    Each instance of the service would need to listen to the _changes feed. Alternatively, a single instance could listen to the feed and post updates to a message queue system (e.g. RabbitMQ) where worker processes could pick up messages to process.

1. _Is CORS supported? Is webhooks supported for data changes?_  
    CORS is supported. Web hooks are not.

1. _What are the deployment options of Cloudant and their differences?_  
    Cloudant can be deployed as Multi-Tenant (MT), Dedicated, or Local. 

    * MT means that cluster resources are shared between different, unrelated customers which may have performance implications. 
    
    * A dedicated deployment means that a cluster is private to you, and sized to your requirements, but still fully managed by Cloudant, 24x7. 
    
    * The Local option means that you can run a Cloudant cluster from within the confines of your own data center, giving you full control, but also the management responsibility.

1. _How are the different options priced?_  
    Cloudant MT is PAYGO based on usage. Dedicated is a flat monthly fee based on the number and spec of nodes. Local is a per-node annual license fee, with a 1-node license suitable for development included in MobileFirst Platform (MFP).

1. _What are the differentiators of Cloudant? What are the competitive advantages of Cloudant against the competitors? Who are they?_  
    Cloudant is a schema-free, distributed, master-less replicating cloud database. Its strongest competitive advantages are its replication capabilities, its durability and its availability. In simplified terms, we can basically yank the power cable of a node in a cluster and the system would be unaffected. Once the node comes back, it would quietly catch up through replication. 

    The NoSQL database space has a large number of different products, all making their particular trade-offs. Such databases (including Cloudant) tend to be more use-case specific than traditional RDBMS. We sometimes encounter MongoDB as a competitor in the field, even though it’s really designed for different goals. Mongo’s storage model is such that it can be seen as more fragile than Cloudant – when Cloudant acknowledges a write, you are guaranteed that the data is written to disk in multiple copies whereas Mongo can be tuned for speed at the expense of durability. 
    
    Other competitors are Amazon’s DynamoDB, and Couchbase.

1. _How would Cloudant be provisioned with data (initial provisioning, regular batch and real time against a backend)?_  
    Cloudant has a simple, easy to use HTTP API for getting data into and out of a cluster. Several IBM ETL tools exist that can be helpful when migrating large data volumes into Cloudant.

1. _How would Cloudant be integrated to the backend?_  
    It depends on the use case and deployment model. Cloudant has a sophisticated ‘changes’ API end-point which provides incremental access to database updates. This can be used to implement various kinds of integrations by using it to incrementally update external systems (e.g., ElasticSearch, external SoR systems). 

    A common pattern we’ve seen is that the master copy of data lives in an existing enterprise system of record database. Cloudant is used as a scalable method to project this data to many mobile devices, either read-only or read-write. In the read-only case, a database trigger or other method must be used to keep the data in Cloudant up to date using Cloudant’s HTTP API. 
    
    If read-write is needed, more sophisticated methods need using as the enterprise system will usually not support incremental sync out-of-the-box. Using Cloudant as the SoR gives the most flexible and easiest-to-manage solution.```

1. _How would Cloudant be managed (initial size and elastic extension, backup etc)?_  
    It depends on the deployment model. Initial size is modeled on the application’s properties: number of users; read vs. write rate; data volumes etc. A typical 3-node dedicated cluster can support of the order of 1,000-1,500 HTTP requests per second. 

    Elastic cluster expansion isn’t fully automatic; growing a cluster requires some operational intervention. For dedicated and MT deployment the management of the cluster is handled by Cloudant (patching, monitoring etc). 
    
    Backups are less important in a replicating, master-less distributed system such as Cloudant as data is stored in triplicate by default. If further backup is required it is straightforward to commission a backup cluster (in a different data center, for example) to which data is replicated continuously. The latter option also provides for snapshot backups. 
    
    More info on [backups](https://ibm.box.com/cloudant-backup-dr)

1. _How is Cloudant be secured (data encryption on device, on Cloudant node etc)_  
    Cloudant’s mobile libraries have the option to encrypt data at rest, and data in flight is always over a secure channel. 

    On a dedicated Cloudant database node, data isn’t encrypted at rest by default, but work is underway to provide an option with full FIPS140-2 compliance on the server. Cloudant Local is of course secured in your own data centers; disk encryption can be used if needed. 
    
    In addition, management of the cluster nodes is restricted to those with an access need and we obviously keep our systems fully patched.

1. _How would Cloudant resolve conflicts on multiple updates on different node within a cluster?_  
    Cloudant has industry-leading conflict handling. 

    Cloudant uses a data model called MVCC, multi-version concurrency control, meaning that each document can exist in several versions. To a certain degree, this is similar to a distributed master-less revision control system such as ‘git’. It means that concurrent modification is a fully supported state of the data, rather than an exception. Finally, no data is ever discarded unless explicitly superseded by business logic. As a consequence, conflict resolution can be deferred to the point where it makes most sense to the application’s business logic.
    
    In a distributed, replicated system the database cannot safely attempt to resolve a data conflict without sacrificing availability. Cloudant provides API to applications using it which support detecting and resolving conflicts.

1. _How would Cloudant synchronize with different DR site? What is the requirement on the network link?_  
    Cloudant is fully set up to do inter-datacenter replication. The draw on the network link depends on many factors: data update frequency, data volume. Traffic goes over a HTTPS link, so all that’s required is a network connection between the two (or more) clusters involved in DR. 

    More DR [info](https://ibm.box.com/cloudant-backup-dr)

1. _How does Cloudant geospatial query?_  
    Geo is a huge topic. 

    Cloudant supports geo-spatial querying and supports GeoJSON, GeoPackage and Web Feature Services. A good introductory overview can be found in the [Cloudant Geo Overview](https://ibm.box.com/cloudant-geo-overview), and details of the geo-spatial API can be found [here.](https://cloudant.com/wp-content/uploads/Cloudant-Geospatial-API-documentation.pdf)

1. _How does Cloudant compare with KONY? KONY claims to offer automatic, rules based conflict resolution and easy sync with traditional RDBMSs._

    KONY is an enterprise mobile app stack similar to IBM MobileFirst. Some technical (but a few years old) information about KONY can be found here:

    http://developer.kony.com/twiki/pub/Portal/Sync/5.0.7/Quick%20Overview%20of%20the%20Kony%20Sync%20Framework.pdf

    On the face of it, KONY doesn't seem to require developers to deal with conflicts and consistency issues like you do in Cloudant. Why can't Cloudant offer something similar?

    KONY makes a different set of trade-offs around the CAP theorem. The KONY server is a non-distributed 'spider in the net' analogous to the MobileFirst Worklight server which talks to an enterprise RDBMS on one side, and the devices on the other. Having a strongly consistent SoR as the source of truth makes certain things easier – and other things hard (or impossible). As the KONY server is (somewhat simplified) a single point, it can define a reliable, strictly increasing sequence (or a time stamp) for updates. Of course, conflicts can still occur between updates from the devices, and updates performed directly on the server, but these can now be resolved through a set of rules (e.g. server always wins, or reject-reload on conflict etc). 

    Cloudant is an AP system without consistency guarantees. In the KONY case, there is a single source of truth. Think of it as a hub-spoke (with the KONY server as the hub) topology. Cloudant's topology is a fully connected set of identical nodes - no single source of truth. In KONY's case, the data store is ACID-compliant: strongly consistent. No conflicts can exist in the data store. In Cloudant, conflicts may exist in the cluster itself as a consequence of replication. 

    In essence, KONY is CP, Cloudant is AP. KONY's weaknesses will be a lack of scalability (all traffic funnelled through the KONY server), a lack of availability (no easy scaleout of the KONY server, as backend is CP). KONY does recognise this, and offers a mode of operation which has a separate 'book keeping' database on the KONY server for cases where the SoR is unavailable – this gives some redundancy in case of network partitions, but offers no real high availability or scalability: it's replacing one CP system with another.

    Could Cloudant resolve conflicts in a similar way to KONY? Yes and no, is the somewhat unhelpful answer. It would be harder to make something for Cloudant that could be driven by a simple, generic rule set that would fit all use cases. However, Cloudant has a comprehensive API for detecting and resolving conflicts. It would be perfectly possible to devise a backend service that would query the cluster for conflicted documents, and apply a business logic based resolution strategy. Cloudant allows you to _defer_ conflict resolution to where it makes the most sense for it to be handled. We recognise that a proper, safe conflict resolution strategy must involve some degree of semantic understanding of the data. A conflict is basically two differing versions of the same record. The database cannot safely divine which is right, or which is wrong.

    Isn't the KONY approach better?

    It is certainly easier to grasp for a developer used to deal with RDBMSs. If this means it's better or not depends on your application, your data, and your budget. If your application relies on strong consistency guarantees, Cloudant is a poor fit as the data store. However, if your application relies on your data always being available, or the cost of downtime is high, Cloudant would be ideal.

    Eventual consistency is hard to grasp in practice, even where developers do understand the concepts. Most developers that are used to RDBMSs would instinctively expect that if you write a record successfully, you can read it immediately and get it back. This cannot safely be relied upon in an eventually consistent database. This is complicated by the fact that in normal operation, or under test conditions the inconsistency window is so short compared with the network latency of the requests that problems may not occur. But it is always there, and greater than zero.






