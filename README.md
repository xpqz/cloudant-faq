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
