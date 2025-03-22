## Forward

The questions require the interviewees to design an architecture for a software system, which could be a news feed, Google search, chat system, etc...

Companies widely adopt system design interviews because the communication and problem-solving skills tested in these interviews are similar to those required by a software engineer's daily work.

This book provides solid knowledge in building a scalable system. The more knowledge gained from reading this book, the better you are equipped in solving the system design questions. 

> [!NOTE]
> Do it instead of just reading the book!!!


## Chapter 1: Scale from zero to Millions of Users

In this chapter, we build a system that supports a single user and gradually scale it up to serve millions of users.

### Single Server Setup

To start with something simple, everything is running on a single server. Figure 1-1 shows the illustration of a single server setup where everything is running on one server.

![[books/figure1-1.png]]In this figure, user (client) send a request to a domain like `api.mysite.com` or `mysite.com`. `DNS` resolves domain's IP address. User getting content of given IP address' content.

Site traffic can come from two different sources to your website:
- Web application: it uses a combination of server-side languages to handle logic,storage,etc... and client-side languages for presentation.
- Mobile application: HTTP protocol is the communication protocol between the mobile app and web server. JSON is commonly used API response format to transfer data due to its simplicity.

### Database

Site traffic increased so one server is not enough for now, we need multiple server. Separating data and web tier to different server so they can be scalable seperatly.

![[books/figure1-2.png]]

#### Which databases to use?
You can choose relation and non-relational databases.

Relational databases also called Relational database management systems (RDBMS) or SQL database.

Non-Relational databases are also called NoSQL databases. These databases are grouped into four categories: 
- Key-Value stores
- Graph stores
- Column stores
- Document stores

If your apps priority is one of them, you can use NoSQL database:
- Your application required super-low latency
- Your data are unstructured, or don't have relational data.
- You only need serialized and deserialize data
- You need to store a massive amount of data.

### Vertical vs Horizontal scaling

Vertical scaling, referred to as "scale up", means the process of adding more power(CPU,RAM,etc...) to your server.
Horizontal scaling, referred to as "scale-out", allows you to scale by adding more server into your pool of resources.

Cons of vertical scaling:
- Has limit. Impossible to add unlimited CPU or memory to a single server.
- Doesn't have fail over and redundancy. If server goes down, the app/site goes down.

### Load balancer (LB)

![[figure1-3.png]]

User's connect to public IP of the load balancer directly. With this setup, web server unreachable directly by clients anymore. 

In [[figure1-3.png]], after load balancer, we successfully added second server. So we solved our fail over issue and improved availability of web tier. More details:
- If server 1 goes offline, all traffic will be routed to server 2.
- If web tier traffic grows, you only need to add more server to server pool and add server's IP to load balancer. LB start to send request to new servers.

The current design, we have one database, so it doesn't support fail over and redundancy. `Database replication` is a common technique to address those problems.

### Database replication

Quote from Wikipedia: "Database replication can be used in many database systems, usually with master/slave relationship between the original(master) and the copies(slaves)".

A master database generally only support write operations(insert, update,delete). A slave database gets copies of the data from master database and only supports read operations(select).

Pros of database replication:
- Better performance: All writes operations happen in master nodes; whereas, read operation are distributed across slave nodes. So we can perform read operations parallel.
- Reliability: If one of your database server is destroyed, you don't need to worry about data loss, because data is replicated across multiple locations.
- High availability: If a database if offline as you can access data stored in another database server.

What happens if one of the databases goes offline?
- If only one slave database is available and it goes offline, read operations will be directed to the master database temporarily. 
- If the master database goes offline, a slave will be promoted to be the new master. In production systems, promoting a new master is more complicated as the data in a slave database might not be up to date. Some other replication methods like multi-masters and circular replication could help.

![[books/figure1-4.png]]

With this architecture, data modyfing operations routing to master, read operations routing to slave database.

We have basic web and data tiers understanding. We have to improve load/response time. We can use cache layer and CDN for this purpose.

### Cache

A cache is a temporary storage area that stores the result of expensive responses or frequently accessed data in memory.

#### Cache tier

The cache tier is a temporary data store layer much faster than the database. Pros of separating cache tier is better performance, reduce database workload and scale of cache independently. 

![[figure1-5.png]] [[figure1-5.png]] shows possible setup of cache server.

Web server first checks cache has the repsonse, if it has, cache returns data to Web server. If not, queries database, stores response to cache and sends to web server.

#### Considerations for using cache
- Consider using cache when data is read frequently but modified infrequently. A cache server is not ideal for persisting data.
- Expiration policy. It is good practice to implement an expiration policy. When there is no expiration policy, cached data will be stored in the memory permanently.
- Consistency: This involves keeping the data store and the cache in sync.
- Mitigation failures: a single cache server represents a potential single point of failure(SPOF). Which means, if one of your component goes down, your entire system goes down.
- Eviction Policy: Once the cache is full, any requests to add items to the cache might cause existing items to be removed. LRU, LFU, FIFO algorithms can be adopted for this purpose.