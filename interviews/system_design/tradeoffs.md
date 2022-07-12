## Trade Offs

### RDBMS - SQL style


### Non RDBMS - Key Value or No-SQL Style
Many services shoping cart, session management, sales rank, product catalog - need only primary key access, using RDBMS would lead to
inefficiencies and limit scale and availability. e.g. Dynamo provides a primary key interface. 
* Dont need complex quering and management functionality provided by RDBMS. The extra functionality requires expensive hardware and skilled personnel
* Also replication technologies chooses consistency over availability

