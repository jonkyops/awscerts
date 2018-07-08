# dynamodb

## APIs

- Table
  - CreateTable – Creates a table and specifies the primary index used for data access.
  - UpdateTable – Updates the provisioned throughput values for the given table.
  - DeleteTable – Deletes a table.
  - DescribeTable – Returns table size, status, and index information.
  - ListTables – Returns a list of all tables associated with the current account and endpoint.
- Item
  - PutItem – Creates a new item, or replaces an old item with a new item (including all the attributes). If an item already exists in the specified table with the same primary key, the new item completely replaces the existing item. You can also use conditional operators to replace an item only if its attribute values match certain conditions, or to insert a new item only if that item doesn’t already exist.
  - UpdateItem – Edits an existing item's attributes. You can also use conditional operators to perform an update only if the item’s attribute values match certain conditions.
  - DeleteItem – Deletes a single item in a table by primary key. You can also use conditional operators to perform a delete an item only if the item’s attribute values match certain conditions.
  - GetItem – The GetItem operation returns a set of Attributes for an item that matches the primary key. The GetItem operation provides an eventually consistent read by default. If eventually consistent reads are not acceptable for your application, use ConsistentRead.
  - BatchWriteItem – Inserts, replaces, and deletes multiple items across multiple tables in a single request, but not as a single transaction. Supports batches of up to 25 items to Put or Delete, with a maximum total request size of 16 MB.
  - BatchGetItem – The BatchGetItem operation returns the attributes for multiple items from multiple tables using their primary keys. A single response has a size limit of 16 MB and returns a maximum of 100 items. Supports both strong and eventual consistency.
- Query/Scan
  - Query –  Gets one or more items using the table primary key, or from a secondary index using the index key. You can narrow the scope of the query on a table by using comparison operators or expressions. You can also filter the query results using filters on non-key attributes. Supports both strong and eventual consistency. A single response has a size limit of 1 MB.
  - Scan – Gets all items and attributes by performing a full scan across the table or a secondary index. You can limit the return set by specifying filters against one or more attributes.
