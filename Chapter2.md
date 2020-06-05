# Chapter 2: Data Models and Query Languages

## Relational Model Versus Document Model
  * Concept for Relational Model: Data is organized into relations (called tables in SQL), where each relation is an unordered collection of tuples (rows in SQL).
### The Birth of NoSQL
  * Several driving forces behind the adoption of NoSQL are:
    * A need for greater scalability than relational databases
    * Preference for free and open source software
    * Specialized query operations not well supported by SQL
    * Frustration with the restrictiveness of relational schemas
### The Object-Relational Mismatch
  * Impedance mismatch: An awkward translation layer (Something like Object-Relational Mapping (ORM) framework) is required between the objects in the application code and the database model of tables, rows and columns.
  * An option is to encode information as a JSON or XML document, store it onto a text column in the database.
  * The JSON/XML representation has better locality than the multi-table schema option, becase JSON/XML makes the one-to-many relationships (tree structure) explicit.
### Many-to-One and Many-to-Many Relationships
  * Approach: having standardized lists of geographic regions and industries (in the example of LinkedIn profile) and let users choose from a drop down list or autocomplete. Advantages:
    * Consistent stype and spelling across profiles
    * Aviod ambiguity
    * Ease of updating
    * Localization support
    * Better search
  * The standardized lists will be a hashmap from ID to text, by doing this, we expect to remove duplication when information (locations or industries) are the same in multiple places. This is called Normalization Many-to-One relationship in the database.
  * Data has a tendency of becoming more interconnected as feature are added to applications, which require many-to-many relationships. The image below illustrates how many-to-many relationship looks like in the LinkedIn profile data model.\
    <img src="./Images/Chapter2/LinkedInManyToMany.png" height=60% width=60%>\
### Are Document Databases Repeating History?
  * How to best represent many-to-many relationships and joins in document databses and NoSQL?
  * The Network Model -- Conference on Data System Languages (CODASYL Model)
    * A record can have multiple parents.
    * Links between records in the network model are more like pointers in a programming language than foreign keys in SQL.
    * An access path -- Follow a path from a root record along these chains of links to access a record.
    * Query in CODASYL -- Move a cursor through the database by iterating over lists of records and follow access path.
  * The Query Model (SQL) -- The query optimizer automatically decides which parts of the query to execute in which order, and which indexes to use.
### Relational Versus Document Databases Today
  * Advantages of both database models
    * For Document Database: Schema flexibity, better performance due to locality, and closer to data structure used by the application. 
    * For Relational Database: Better support for joins, many-to-one and many-to-many relationships.
  * Which data model leads to simpler application code?
    * Better to use Document Database if the data in the application has a document-like structure in order to avoid shredding (splitting a document-like structure into multiple tables).

## Query Languages for Data

## Graph-Like Data Models