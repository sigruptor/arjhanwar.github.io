# System Design Concepts

Few important characteristics:
1. Availability
2. Reliability
3. Scalaibility
4. Efficiency -> Latency/Throughput
5. Maintainability/Servicability

### Scalability
It is the capability of a system, network or process to grow and manage increased demand. A system needs to scale because of either increase in data volume 
or increase in amount of work. The target is to be able to achieve this scaling without performance loss.

#### Horizontal Scaling - Clones
Adding more servers/machines to the pool of resources. All these servers usually sit behind LB and client would (should) not see any impact as LB now has
many more servers to send the request to. Mostly means stateless services (or a state is stored somewhere in a DB). When a new service/server comes up,
it picks up the state from DB and starts processing subsequent requests.

But servers ultimately cant scale beyond a point, and the limit becomes something else e.g. may be a DB (MySQL)

##### Scaling - Databases
_**Relational Databases**_: MySql, PostGres
- Each row contains information about one entity and each column contains separate entities.
- MySql with more power (more CPU, memory)
- Add Sharding, Denormalization or SQS Tuning
- But there is a limit beyond which it is quite expensive to scale these.

_**Non Relational - No SQL**_:</br>
**1. Key-Value Store**: data is stored in array of key-value stores. Dynamo, S3 (object storage -> image dataset, video dataset), Redis, Voldemort</br>
**2. Columnar Store:** Cassandra or HBase. Have column families which are container of rows. Dont store all values of a row together, rather store values 
from each column together. Each column family could be a file, so we just need to query the file with relevant columns. 
- Can do columnar compression, encoding or bloom filters for querying </br>
**3. Document store**: Data is stored in documents and these documents are grouped together in collections. Each document can entirely diff structure e.g. MongoDb </br>
**4. Graph Store: Node:** Store data whose relations are best respresented by graph. Nodes (entities, properties) and lines (relationship b/w entities). e.g. Neo4j</br>

| Type      | SQL | NO-SQL    |
| :---        |    :----   |          :--- |
| Storage     | Row has the data and column stores attributes       | Diff types (Key-Value, Columnar etc.) mostly following key-object notation  |
| Schema      | has a fixed schema, each row to contain all columnar values as well  | No fixed schemas, attributes (columns) can be added at an individual key or object level|
| Querying    |  SQL | Unstructured Query Languageg
| Scalability    | Vertically scalable | Horizontally scalable
| Reliability    | ACID compliant - transactional based | Sacrifice ACID for scalablity and performance


Use Relatinal databases: 
- When ACID compliance or financial information (Transactional based systems)
- Data is structured and unchanging
- E.g. CockroachDB, Google Spanner

Use NoSql Databases:
- Rapid development: When no structure and allows us to add more types. We can make quick iterations on systems which require frequent update to Data structures.
- Cloud Storage

_**NOTE:**_
**Obviously with noSQL databases also, we need to manage relationship or joins at some level, the general recommendation is to do in application code. **

Now that we have a fully functional database, we are able to scale and serve all the clients. But the problem is, the system is slow whenever service has
to fetch data from Cache.

#### Caching
In memory cache solutions - Memcache (scales very easily )or Redis (Redis also provides persistent layer)

**1. Caching the query**: We can cache the complex query (as key) and result as the value. This is not so nice, everytime there is an update to even one cell, we need to discard and update the query result again. Also it is hard to delete a complex result, when a particular object expires.
**2. Caching the object**: Cache the object itself, easy to update/discard.
Some ideas of objects to cache:
- user sessions (never use the database!)
- fully rendered blog articles
- activity streams
- user<->friend relationships


#### Asynchronism
Style 1: You do the precomputation in advance (may be at night, or regularly via a cron job) and when the user requests comes, that can be served. e.g. Converting dynamic content of website into html pages on every change. </br>
Style 2: When a user submits a job e.g. a compute intensive task, the task can be added to the queue and worker queue can handle it. While this task is handled by the backend, user should still be able to navigate and do other things rather than to just wait and sit idle. </br>

