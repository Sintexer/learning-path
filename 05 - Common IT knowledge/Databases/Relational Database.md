A relational database organizes data into **tables**. Data in one table can *link* to data in other tables to create **relationships** - hence, the relational part of the name.  
  
A table stores data in **rows** and **columns**. A row, often called a record, contains all information about a specific entry. Columns describe attributes of an entry. 

The tables, rows, columns, and relationships between them is called a logical schema. With relational databases, a schema is fixed. After the database is operational, it becomes difficult to change the schema. Because of this, most of the data modeling is done up front before the database is active.

### Relation databases benefits

1. Complex [[SQL]] queries.
2. Reduced redundancy - through data normalization.
3. Familiarity - most people have experience.
4. Accuracy - [[ACID]] principle

## Use cases

- **Applications with fixed schema**
- **Applications that need persistent storage** and follow the ACID principle.