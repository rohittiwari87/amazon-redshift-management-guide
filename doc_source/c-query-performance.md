# Factors Affecting Query Performance<a name="c-query-performance"></a>

A number of factors can affect query performance\. The following aspects of your data, cluster, and database operations all play a part in how quickly your queries process\.
+ **Number of nodes, processors, or slices** – A compute node is partitioned into slices\. More nodes means more processors and more slices, which enables your queries to process faster by running portions of the query concurrently across the slices\. However, more nodes also means greater expense, so you will need to find the balance of cost and performance that is appropriate for your system\. For more information on Amazon Redshift cluster architecture, see [Data Warehouse System Architecture](c_high_level_system_architecture.md)\. 
+ **Node types** – An Amazon Redshift cluster can use either dense storage or dense compute nodes\. The dense storage node types are recommended for substantial data storage needs, while dense compute node types are optimized for performance\-intensive workloads\. Each node type offers different sizes and limits to help you scale your cluster appropriately\. The node size determines the storage capacity, memory, CPU, and price of each node in the cluster\. For more information on node types, see [Amazon Redshift Pricing](https://aws.amazon.com/redshift/pricing/)\.
+ **Data distribution** – Amazon Redshift stores table data on the compute nodes according to a table's distribution style\. When you execute a query, the query optimizer redistributes the data to the compute nodes as needed to perform any joins and aggregations\. Choosing the right distribution style for a table helps minimize the impact of the redistribution step by locating the data where it needs to be before the joins are performed\. For more information, see [Choosing a Data Distribution Style](t_Distributing_data.md)\. 
+ **Data sort order** – Amazon Redshift stores table data on disk in sorted order according to a table’s sort keys\. The query optimizer and the query processor use the information about where the data is located to reduce the number of blocks that need to be scanned and thereby improve query speed\. For more information, see [Choosing Sort Keys](t_Sorting_data.md)\. 
+ **Dataset size** – A higher volume of data in the cluster can slow query performance for queries, because more rows need to be scanned and redistributed\. You can mitigate this effect by regular vacuuming and archiving of data, and by using a predicate to restrict the query dataset\. 
+ **Concurrent operations** – Running multiple operations at once can affect query performance\. Each operation takes one or more slots in an available query queue and uses the memory associated with those slots\. If other operations are running, enough query queue slots might not be available\. In this case, the query will have to wait for slots to open before it can begin processing\. For more information about creating and configuring query queues, see [Implementing Workload Management](cm-c-implementing-workload-management.md)\. 
+ **Query structure** – How your query is written will affect its performance\. As much as possible, write queries to process and return as little data as will meet your needs\. For more information, see [Amazon Redshift Best Practices for Designing Queries](c_designing-queries-best-practices.md)\. 
+ **Code compilation** – Amazon Redshift generates and compiles code for each query execution plan\. The compiled code segments are stored in a least recently used \(LRU\) cache and shared across sessions in a cluster\. Thus, subsequent executions of the same query, even in different sessions and often even with different query parameters, will run faster because they can skip the initial generation and compilation steps\. The LRU cache persists through cluster reboots, but is wiped by maintenance upgrades\. 

  The compiled code executes faster because it eliminates the overhead of using an interpreter\. You always have some overhead cost the first time code is generated and compiled\. As a result, the performance of a query the first time you run it can be misleading\. The overhead cost might be especially noticeable when you run one\-off \(ad hoc\) queries\. You should always run a query a second time to determine its typical performance\.

  Similarly, be careful about comparing the performance of the same query sent from different clients\. The execution engine generates different code for the JDBC connection protocols and ODBC and psql \(libpq\) connection protocols\. If two clients use different protocols, each client will incur the first\-time cost of generating compiled code, even for the same query\. Other clients that use the same protocol, however, will benefit from sharing the cached code\. A client that uses ODBC and a client running psql with libpq can share the same compiled code\.