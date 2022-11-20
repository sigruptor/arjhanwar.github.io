# Data Models & Query Languages
Most applications are built by layering one data model on top of other.
- App developer models it in terms of business logic and ApIs that manipulate those data structures.
- When we store those data structures, its expressed in JSON or XML, tables in a relational database or a graph model.
- Engineers who built the database software have a different representation in terms of bytes in memory, disk or network. It allows the data to be queried, searched, manipulated and processed.
- Lower levels hardware engineers represent bytes in terms of currents.

### Relational Model vs Document Model
SQL - data organized into relations (called tables in SQL), it was designed with original scope of business data processing, but generalized pretty well
for online publishing, discussion, social networking, ecommerce, games etc.

NOSQL - misnomer as it does not represent any particular technology.
- Need for greater scalability including very large datasets and high write throughput.
- Widespread preference for free and open source software.
- Specialized query operations that are not well supported in relational databases
- A desire for more dynamic and expressive data model getting away for rigid schemas.

#### Object Relation Mismatch
Case Study : Modelling a resume, a resume could be modelled in a relational table, with multiple tables (users, organization, titles, education etc.), where it would
have many-many relationships among them. A query to get data would require a multi-join between them or multiple queries.
Now, this could also be modelled into a document store with a hierarchy. e.g. user-> (positions -> (job1 - (title1, company1), job2 - (title2, company2)), education ...)

JSON model has better locality but a lot of repeatition. If an application requires all of the data of the user together then this model works good.
HOwever, it  has repeatition, if we want to change the org name, entire DB has to be traversed (making it expensive)

### Relational Versus Document Databases today
**Document**: - schema flexibility
- Better performance due to locality of data
- for some applications it is closer to the data structure used by application.

**Relational** - better support for joins
- Many-to-one relationship and many-to-many relationship

#### Application Code
One aspect is which model leads to a simpler application code. Obviously it depends on the type of application. e.g. if data in application has a document
like structure i.e. a tree of one-to-many relationships, where entire tree is loaded at once, then its probably a good idea to use _document model_. Relational
technique of shredding - splitting a document like structure into multiple tables can lead to combersome schemas and unnecessary complicated code at application layer.
- Though we cant refer to deeply nested field directly (not a problem is nesting is not too deep)
- For even reading one field, we would have to load entire document in memory 
- For changing one part of document, it requires a rewrite of entire doc

However for cases when many-many relationship joins are not needed, it still makes a good case. Its possible to reduce number of joins by denormalizing,
but that requires application to make sure data stays consistent across tables.
- Application becomes complex and slower as well as a join performed by app is slower than specialzed code inside database. For highly connected data,relational models are acceptable

#### Schema Flexibility
Document model provides a lot of flexibility wrt schema, you can add a field, remove it. Application code can handle all of it and database model need not change
at all.
**Schema-on-Read**__: Strucuture of data is implicit and only interpreted when its read in contrast with schema-on-write (traditional approach of relational
database where schema is explicity and database ensures all written data conforms to it.
- When you have to change something, just change it in code. However for relational you have to change the data model & perform migration. Schema migration 
though have a bad repuation of being slow and downtime, most relational dbs run `Alter` command within seconds, they dont copy the entire data and smartly just adds/removes a field.
- Approach is useful when data being read dont have a fixed schema or is heterogeneous.
- Structure of data is suspectible to change.

#### Data Locality for Queries
A document is stored as a single continous stream/string string (json, xml), if application needs entire document, there is performance advantage of locality.
If data is split across multiple tables, multiple index lookups are required to retrieve it all, which may require more disk seeks and take more time.
- This advantage applies only if you need the entire document together. If you only need a portion, you would still be loading the entire document.
- Similarly for uploads/updates, if you are only modifying a portion, replacing the entire document is an overkill.

Idea of grouping related data together for locality is not limited to document model anymore. Spanner offers same locality model, oracle allows it with multi
-table cluster tables, column family concept in Bigtable.

#### Convergence
Now relational database supports JSON/XML as well, this includes ability to index and query inside XML documents.
On the documents side, REthinkDB supports relational like joins.

### Query Languages for Data
#### Imperative Languages
Some languages are imperative like Java. We define the program instruction by instruction, system has to adhere to user defined function and execute it 
instruction by instruction. 
- Tells the computer to perform certain operations in a certain order
- e.g. Java

#### Declarative Languages
**Pattern of execution**
- We need to just specify the goal (data you want), not have to worry about how to acheive that goal.
- Just define the pattern of data - what conditions the result must meet and how the data is to be transformed (sorted, grouped), but not the mechanism.
- e.g. SQL
- Its more concise and hides the implementation details of database engine, which makes it possible for database to do optimizations/improvements without
any changes to queries.
- System has to be defined/specialized/optimized once for queries and can be used across diff queries.

**Internal representation**
- this allows database more freedom to move data internally, compact it etc.

**Execution**
- They internally maps the query to execute parallely,
- Declarative languages have better chance of getting faster in parallel execution because they control the algorithm to determine those results.


